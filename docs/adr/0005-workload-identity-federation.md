# ADR-0005 · Workload Identity Federation GCP↔GitHub

- **Status**: Proposed
- **Date**: 2026-05-09
- **Scope**: all (aplica a apps con deploy CI/CD a GCP)
- **Implements**: D-005

## Context

Las apps de categoría `web-app` y `server-only` que despliegan a Cloud Run o GKE necesitan que GitHub Actions tenga credenciales GCP para hacer `gcloud builds submit`, `gcloud run deploy`, push a Artifact Registry, etc.

El patrón histórico era Service Account Keys: archivo JSON de larga duración pegado como secret en GitHub Actions (`GCP_SA_KEY`). Problemas conocidos:

- Si el JSON se filtra (logs accidentales, fork malicioso, dump de variables), el atacante tiene acceso permanente hasta rotación manual.
- Rotación manual es operacionalmente costosa y se posterga.
- Google ha marcado SA Keys como **fuertemente desaconsejado** desde 2024.

Workload Identity Federation (WIF) es la alternativa moderna estándar:

- GitHub Actions obtiene un token OIDC firmado por GitHub (corta duración, contexto inmutable: repo, ramo, env).
- Google lo cambia por credenciales GCP de corta duración (1h por defecto).
- Sin secret persistente que filtrar.

## Decision

**Adoptar Workload Identity Federation GCP↔GitHub para todas las apps de la suite que despliegan a GCP (Cloud Run, GKE, Artifact Registry).**

Configuración estándar:

- Cada proyecto GCP de Capa 1 (`atlax-ai-dev`, `atlax-ai-pre`, `atlax-ai-pro`) tiene un **Workload Identity Pool** llamado `github-actions`.
- Cada app tiene un **Provider** dentro del pool: `<app>-deploy`.
- Cada provider impersona una **Service Account** dedicada con los permisos mínimos para deploy.
- Restricción `attribute.repository == "atlax-360-ai-suite/<app>"` para evitar que cualquier repo arbitrario pueda obtener credenciales.

**Alcance limitado a CI/CD deploy pipelines** (GitHub Actions). No aplica a:

- `edge-tooling` (CLIs locales, hooks de Claude Code): siguen usando `gcloud auth login` local + SA keys en `chmod 600` cuando sean estrictamente necesarias.
- Apps en Vercel: usan Vercel's OIDC integration con GCP (mismo patrón conceptual) — fuera del scope formal de este ADR pero recomendado.

## Alternatives Considered

### Alternativa A — Service Account Keys en GitHub Secrets

El patrón histórico.

- **Pro**: setup trivial (10 min con `gcloud iam service-accounts keys create`).
- **Contra**: el SA key dura años si no se rota; basta una filtración para tener un atacante persistente; Google lo marca como obsoleto.
- **Descartada**: incompatible con security baseline 2026.

### Alternativa B — `gcloud auth print-access-token` desde un runner self-hosted

Tener un runner GitHub Actions self-hosted en GCP con metadata server.

- **Pro**: sin keys, sin secrets.
- **Contra**: añade operación de runner (parches, actualización, downtime); más caro a escala que GitHub-hosted; menos auditabilidad.
- **Descartada**: para una suite de 5+ apps el coste operacional excede el beneficio.

### Alternativa C — OIDC con AWS y federación cross-cloud

Solución vendor-neutral pero compleja.

- **Pro**: portable si la suite migra a AWS.
- **Contra**: dos federaciones que mantener; latencia extra; más superficie de ataque.
- **Descartada**: la suite está estandarizada en GCP por D-008 + Vercel.

## Consequences

### Positivas

- **Cero secrets de larga duración** en GitHub Actions para deploys GCP.
- **Tokens efímeros** (1h default) — ventana de exposición mínima.
- **Auditoría granular en Cloud Audit Logs**: cada deploy es trazable a `repo:atlax-360-ai-suite/<app>:ref:refs/heads/main:sha:abc123`.
- **Revocación instantánea**: si comprometen un repo, basta editar la condición del Provider para revocar el acceso (sin esperar rotación de keys).

### Negativas

- **Setup inicial más complejo** que SA keys (~30 min por app: crear pool, provider, SA, bindings IAM).
- **Curva de aprendizaje**: devs nuevos necesitan entender cómo Google verifica el token OIDC de GitHub.
- **Dependencia de la disponibilidad de OIDC en GitHub**: si GitHub OIDC tiene outage, los deploys se bloquean. Mitigación: rollback de SA keys "break-glass" guardadas offline para emergencias.

### Neutras

- Documentar el patrón canónico en `template-ai-app` (cuando se cree) ahorra el setup repetitivo.

## References

- [SPEC.md §4.4 — Identidad GCP↔GitHub](../SPEC.md#44-identidad-gcpgithub)
- [SPEC.md §3.1 — Categorías de app (WIF aplica a web-app y server-only)](../SPEC.md#31-categorías-de-aplicación)
- [GCP — Workload Identity Federation docs](https://cloud.google.com/iam/docs/workload-identity-federation)
- [GitHub — OIDC with GCP](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-google-cloud-platform)
