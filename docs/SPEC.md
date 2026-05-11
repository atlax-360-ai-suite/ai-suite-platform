# Atlax 360 AI Suite — Patrón "Shared Platform"

- **Status**: Accepted v0.6
- **Owner**: jgcalvo@atlax360.com
- **Date**: 2026-05-09 | **Última revisión**: 2026-05-11
- **Scope**: cross-project (`atlax-ai`)
- **Aplica a**: kairos, atlax-claude-dashboard, atlax-langfuse-bridge, atlax-observatorios, harvest, futuras apps de la categoría
- **Repo canónico**: [atlax-360-ai-suite/ai-suite-platform](https://github.com/atlax-360-ai-suite/ai-suite-platform)
- **Estado**: documento de referencia compartido — cualquier proyecto puede proponer cambios vía PR a este repo

### Changelog

| Versión | Fecha      | Cambios                                                                                                              |
| ------- | ---------- | -------------------------------------------------------------------------------------------------------------------- |
| v0.1    | 2026-05-09 | Draft inicial generado desde Kairos. 5 capas, 8 decisiones D-001..D-008                                              |
| v0.2    | 2026-05-09 | Consolidación con reporte langfuse-bridge: §3.1 categorías, leyenda §11, KP-01..KP-05 aplicados                      |
| v0.3    | 2026-05-09 | Corrección dominio: `atlax360.ai` canónico (no `atlax.ai`). D-002 actualizada a Accepted. D-009 nueva.               |
| v0.4    | 2026-05-10 | Movido a repo dedicado `atlax-360-ai-suite/ai-suite-platform`. Ya no es host transitorio en Kairos. §14 actualizado. |
| v0.4.1  | 2026-05-11 | ADRs Michael Nygard ADR-0001..ADR-0010 formalizando D-001..D-010 en `docs/adr/`. §10 con columna ADR.                |
| v0.5    | 2026-05-11 | D-011 nueva: email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD. §9.6 nueva. ADR-0011.              |
| v0.6    | 2026-05-11 | D-012 nueva: `@atlax/auth` postponed (insufficient consumer overlap). ADR-0012.                                      |

---

## 1. Propósito

Definir el **patrón de plataforma compartida** para la categoría de aplicaciones "Atlax 360 AI Suite" — apps que mejoran la productividad de Atlax o integran IA, algunas internas, otras potencialmente comercializables.

El objetivo es **separar lo que cambia rápido (apps) de lo que cambia lento (plataforma)** y evitar acoplamientos que duelan en el año 2.

Este documento es la especificación de referencia. Cada proyecto valida sus hipótesis contra él y propone ajustes vía PR a este repo.

## 2. Principios rectores

1. **Una identidad, muchas apps**: una sola Consent Screen "Atlax 360 AI Suite", N OAuth Clients (uno por app)
2. **Plataforma estable, apps volátiles**: lo horizontal (auth, DNS, observability) en proyectos GCP shared; lo vertical (BD, deploys) por app
3. **Repo por app**: cada app es independiente — repo, ciclo de vida, equipo, ritmo
4. **Subdominios canónicos**: `<app>.atlax360.ai` para producción; `<app>.<env>.atlax360.ai` para entornos no productivos
5. **CI/CD homogéneo**: template repo con workflows comunes; `bun` como runtime; Conventional Commits
6. **Defer cuando puedas**: no construir abstracciones hasta que haya 2+ casos reales que las justifiquen

## 3. Arquitectura por capas

```
┌─────────────────────────────────────────────────────────────────┐
│ Capa 1 — Plataforma compartida (cambia poco, pocos owners)     │
│  • 3 proyectos GCP: atlax-ai-{dev,pre,pro}                      │
│  • OAuth Consent Screen "Atlax 360 AI Suite" (en pro)           │
│  • DNS zone atlax360.ai (DonDominio)                            │
│  • Workload Identity Federation GCP↔GitHub                      │
│  • Logs sink central + Secret Manager compartido                │
└─────────────────────────────────────────────────────────────────┘
                                ↓ identidad
┌─────────────────────────────────────────────────────────────────┐
│ Capa 2 — OAuth Clients (uno por app, todos bajo misma Consent)  │
│  • Client "Kairos"        + redirect URIs (local/dev/pre/pro)   │
│  • Client "Dashboard"     + redirect URIs                       │
│  • Client "Langfuse-bridge" + redirect URIs                     │
└─────────────────────────────────────────────────────────────────┘
                                ↓ credentials
┌─────────────────────────────────────────────────────────────────┐
│ Capa 3 — Aplicaciones (repo + Vercel/Cloud Run + Supabase/DB)  │
│  • github.com/atlax/kairos       → vercel/kairos       → SB pro │
│  • github.com/atlax/dashboard    → vercel/dashboard    → CR pro │
│  • github.com/atlax/<futura>     → ...                          │
└─────────────────────────────────────────────────────────────────┘
                                ↓ runtime
┌─────────────────────────────────────────────────────────────────┐
│ Capa 4 — CI/CD homogéneo (template repo "template-ai-app")     │
│  • .github/workflows/ci + deploy-{dev,pre,pro}.yml              │
│  • .claude/CLAUDE.md heredable                                  │
│  • packages/@atlax/auth (cliente Supabase + Google OAuth)       │
│  • scripts/ops/preflight.sh genérico                            │
└─────────────────────────────────────────────────────────────────┘
                                ↓ (futuro)
┌─────────────────────────────────────────────────────────────────┐
│ Capa 5 — Portal unificador (apps.atlax360.ai, NO urgente)       │
│  • Single login con cookie *.atlax360.ai                        │
│  • Launcher cross-app                                           │
│  • Construir cuando ≥3 apps con usuarios solapados              │
└─────────────────────────────────────────────────────────────────┘
```

### 3.1 Categorías de aplicación

No todas las apps del ecosistema son aplicaciones web con servidor HTTP. Tres categorías, con distintos requisitos:

| Categoría        | Descripción                                                                          | Ejemplos                                          |
| ---------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------- |
| **web-app**      | Tiene servidor HTTP, UI o API. Responde peticiones en tiempo real.                   | Kairos, atlax-claude-dashboard, harvest           |
| **edge-tooling** | CLIs, hooks, scripts. Corren en máquinas dev / CI sin servidor HTTP permanente.      | atlax-langfuse-bridge, hooks de Claude Code       |
| **server-only**  | Backend persistente sin UI pública. APIs internas, workers, cron jobs, long-running. | Futuros workers de embedding, ingesta batch, etc. |

**Aplicabilidad de invariantes por categoría:**

| Invariante                      | web-app | edge-tooling   | server-only    |
| ------------------------------- | ------- | -------------- | -------------- |
| Auth-first middleware           | ✅      | ⊘ N/A          | ✅             |
| CORS allowlist                  | ✅      | ⊘ N/A          | ✅ (si HTTP)   |
| CSP obligatorio                 | ✅      | ⊘ N/A          | ⊘ N/A          |
| Health endpoint GET /api/health | ✅      | ⊘ N/A          | ✅ (si HTTP)   |
| Correlation ID X-Request-ID     | ✅      | 🟡 best-effort | ✅             |
| Logs JSON estructurados         | ✅      | ✅             | ✅             |
| AbortSignal.timeout en fetch    | ✅      | ✅             | ✅             |
| Retry exponential backoff       | ✅      | ✅             | ✅             |
| Workload Identity Federation    | ✅ (CI) | ⊘ N/A          | ✅ (CI)        |
| OAuth Client (Google)           | ✅      | ⊘ N/A          | ⊘ según uso    |
| Subdomain `<app>.atlax360.ai`   | ✅      | ⊘ N/A          | 🟡 si expuesto |

**Leyenda**: ✅ aplica | 🟡 aplica parcialmente / best-effort | ❌ no cumple (deuda) | ⊘ no aplica por diseño

## 4. Capa 1 — Plataforma compartida

### 4.1 Proyectos GCP

**3 proyectos** en la organización GCP de Atlax:

| Proyecto       | Propósito                                                         | Quién accede                           |
| -------------- | ----------------------------------------------------------------- | -------------------------------------- |
| `atlax-ai-dev` | Recursos compartidos para entornos de desarrollo de cualquier app | Ingenieros                             |
| `atlax-ai-pre` | Recursos compartidos para staging / preview                       | Ingenieros + QA                        |
| `atlax-ai-pro` | Recursos compartidos para producción                              | Solo despliegue automatizado + on-call |

> **Importante**: estos proyectos NO contienen código de aplicaciones. Son sustrato. Cloud Run/GKE/Supabase de cada app pueden vivir aquí o en proyectos separados según conveniencia.

### 4.2 OAuth Consent Screen

**Una sola** Consent Screen viviendo en `atlax-ai-pro`:

- **App name**: `Atlax 360 AI Suite`
- **User type**: `Internal` mientras todas las apps sean internas; `External + verified` cuando se comercialice alguna
- **Authorized domains**: `atlax360.ai`, `atlax360.com`, `supabase.co`
- **Privacy policy / Terms**: URL única `https://atlax360.ai/legal/privacy` y `terms`

**Por qué una sola**: el usuario ve consistencia de marca, una sola review de Google si se va a External, una sola política legal a mantener.

### 4.3 DNS

Zona `atlax360.ai` gestionada en DonDominio. Subdominios:

| Patrón                             | Uso                                                   |
| ---------------------------------- | ----------------------------------------------------- |
| `<app>.atlax360.ai`                | Producción                                            |
| `<app>.pre.atlax360.ai`            | Pre/staging                                           |
| `<app>.dev.atlax360.ai`            | Dev (opcional — preview URLs de Vercel suelen bastar) |
| `apps.atlax360.ai` o `atlax360.ai` | Portal unificador (futuro)                            |

**Estado actual**: Kairos vive en `kairos.atlax360.ai`. Conforme se añadan nuevas apps de la suite, sus subdominios se registran directamente en DonDominio bajo el mismo dominio.

### 4.4 Identidad GCP↔GitHub

**Workload Identity Federation** en cada proyecto GCP. Evita Service Account keys de larga duración. GitHub Actions obtiene un token OIDC y lo cambia por credenciales GCP de corta duración. Patrón estándar 2026.

> **Alcance**: WIF aplica únicamente a **CI/CD deploy pipelines** (GitHub Actions que despliegan a Cloud Run, GKE, etc.). No aplica a herramientas de tipo `edge-tooling` (CLIs, hooks de Claude Code, scripts de dev) — esas usan credenciales locales de `gcloud auth login` o service account keys en fichero local con `chmod 600`. Ver categorías en §3.1.

### 4.5 Observability

Sink central de logs (Loki, Datadog, Better Stack o similar) en `atlax-ai-pro`. Cada app envía logs estructurados (JSON) con campos canónicos:

- `service` — nombre de la app (kairos, dashboard, langfuse-bridge)
- `environment` — dev/pre/pro
- `requestId` — correlation ID propagado E2E
- `traceparent` — W3C trace context

## 5. Capa 2 — OAuth Clients

### 5.1 Política

- **Un OAuth Client por app**, nombrado igual que la app (`Kairos`, `Dashboard`, `Langfuse-bridge`)
- Todos cuelgan de la misma Consent Screen
- Cada client tiene **redirect URIs para todos los entornos** (local/dev/pre/pro) — un solo client cubre el ciclo de vida completo
- **Client Secrets en Secret Manager** del proyecto GCP correspondiente, no en `.env.local` ni en repos

### 5.2 Por qué un client por app

- **Aislamiento de blast radius**: filtras un secret → revocas solo esa app
- **UX clara**: usuario ve "Kairos" en `myaccount.google.com → Apps with access`
- **Permisos evolucionables**: cada app pide los scopes que necesita (Drive, Calendar, etc.) sin afectar al resto
- **Auditoría granular**: logs de OAuth por app

### 5.3 Por qué una sola Consent Screen

- Branding consistente
- Una sola verificación de dominios
- Una sola review de Google si pasa a External
- Una sola política de privacidad y términos

## 6. Capa 3 — Aplicaciones

### 6.1 Estructura por app

Cada app es la tripleta:

```
github.com/atlax/<app>
   ↓ deploys to
vercel/<app>  o  cloud-run/<app>
   ↓ uses
supabase/<app>-{dev,pre,pro}  o  cloud-sql/<app>
```

### 6.2 Stack canónico (recomendado, no obligatorio)

| Capa          | Tecnología                           | Notas                                      |
| ------------- | ------------------------------------ | ------------------------------------------ |
| Runtime       | Bun                                  | Nunca npm/yarn/npx                         |
| Lenguaje      | TypeScript strict                    | `^5.9.3` o superior                        |
| Web framework | Next.js 16 + React 19 (App Router)   | Cuando aplica                              |
| API framework | Hono ^4.12 sobre Fluid Compute       | NO Edge Functions                          |
| AI provider   | Anthropic via Vercel AI Gateway      | Default                                    |
| AI SDK        | Vercel AI SDK v6                     | `generateObject` + Zod                     |
| ORM           | Drizzle ORM                          | Sobre Postgres                             |
| BD            | Supabase (Postgres + Auth + Storage) | RLS obligatorio multi-tenant               |
| Validación    | Zod 4                                | NO Zod 3                                   |
| CSS           | Tailwind v4                          | Con `tw-animate-css`                       |
| UI            | shadcn/ui + Radix                    | Sobre `@atlax/design-system` cuando exista |
| Observability | pino → log drain central             | JSON estructurado                          |
| Deploy web    | Vercel                               | Repo conectado a GitHub                    |
| Deploy API    | Vercel Fluid Compute / Cloud Run     | Según latencia/coste                       |

Apps con stack diferente (PHP/Symfony, Python/FastAPI) son legítimas si el caso de negocio lo justifica — pero deben implementar los **invariantes transversales** (sección 9).

### 6.3 Repo layout recomendado

```
<app>/
├── apps/
│   ├── web/             # Next.js (si aplica)
│   └── api/             # Hono (si aplica)
├── packages/
│   ├── @<app>/db/       # Drizzle schema + queries
│   ├── @<app>/types/    # tipos compartidos
│   ├── @<app>/ui/       # componentes propios
│   └── @<app>/...       # módulos verticales
├── docs/
│   ├── 00-INDEX.md
│   ├── adr/             # ADRs Michael Nygard
│   └── ...
├── scripts/ops/         # preflight, deploy, env vars docs
├── e2e/                 # Playwright
├── CLAUDE.md            # convenciones del proyecto
└── package.json         # bun workspaces
```

### 6.4 Subdominios

| App                    | Subdominio canónico (atlax360.ai) | Estado           |
| ---------------------- | --------------------------------- | ---------------- |
| Kairos                 | `kairos.atlax360.ai`              | ✅ activo        |
| atlax-claude-dashboard | `dashboard.atlax360.ai`           | 🟡 por confirmar |
| atlax-langfuse-bridge  | `langfuse.atlax360.ai`            | ✅ activo        |
| atlax-observatorios    | `observatorios.atlax360.ai`       | 🟡 por confirmar |
| harvest                | `harvest.atlax360.ai`             | 🟡 por confirmar |

> **edge-tooling** como `atlax-langfuse-bridge` puede no tener subdominio público propio si sirve como backend interno. El subdominio aplica solo al endpoint de Langfuse UI / API que los usuarios acceden directamente.

## 7. Capa 4 — CI/CD homogéneo

### 7.1 Template repo

`github.com/atlax-360-ai-suite/template-ai-app` — repo template con todo lo común:

```
template-ai-app/
├── .github/workflows/
│   ├── ci.yml              # bun install + test + typecheck + lint en cada PR
│   ├── deploy-dev.yml      # auto en push a main: vercel --target=preview
│   ├── deploy-pre.yml      # auto en tag v*-rc: vercel --prod --target=staging
│   └── deploy-pro.yml      # auto en tag v* (no pre-release): vercel --prod
├── .claude/
│   ├── CLAUDE.md           # convenciones Atlax-AI heredables
│   └── settings.json       # hooks/permisos comunes
├── packages/
│   └── @atlax/auth/        # cliente Supabase + Google OAuth pre-configurado
├── scripts/ops/
│   ├── preflight.sh        # genérico, parametrizado por env
│   └── deploy.sh
├── vercel.ts               # config Vercel canónico (vercel.ts, no vercel.json)
├── bunfig.toml
└── tsconfig.json
```

Crear app nueva = `gh repo create atlax-360-ai-suite/<name> --template atlax-360-ai-suite/template-ai-app` + 5 minutos de variables de entorno.

### 7.2 Ciclo Local → DEV → PRE → PRO

```
Local (bun dev)
   ↓ push to feature branch
Pull Request → CI (lint, test, typecheck, e2e)
   ↓ merge to main
Auto-deploy DEV (vercel preview o <app>.dev.atlax360.ai)
   ↓ tag v*-rc
Auto-deploy PRE (<app>.pre.atlax360.ai)
   ↓ tag v* (no pre-release)
Auto-deploy PRO (<app>.atlax360.ai)
```

**Reglas:**

- `main` siempre desplegable
- Conventional Commits estricto: `feat`, `fix`, `refactor`, `audit`, `test`, `docs`, `improve`, `ops`
- Squash merge desde feature branches
- Tags semver: `v1.2.3` para PRO, `v1.2.3-rc.1` para PRE
- Rollback: `vercel rollback` (web) o `gcloud run services update-traffic` (API)

### 7.3 Branch protection

- `main`: branch protection ON; PR obligatoria; CI verde obligatorio; sin force-push
- Sin commits directos a `main` — esto aplica también a Claude Code (regla global ya en CLAUDE.md)

## 8. Variables de entorno — convenciones

### 8.1 Naming

| Prefijo         | Significado                                                 | Ámbito          |
| --------------- | ----------------------------------------------------------- | --------------- |
| `<APP>_*`       | Específica de la app (`KAIROS_*`, `DASHBOARD_*`)            | Solo esa app    |
| `ATLAX_*`       | Compartida por toda la suite                                | Plataforma      |
| `NEXT_PUBLIC_*` | Expuesta al cliente                                         | Solo lo público |
| Sin prefijo     | Estándar de la industria (`DATABASE_URL`, `OPENAI_API_KEY`) | Cualquier app   |

### 8.2 Inventario obligatorio

Cada repo mantiene `scripts/ops/PRO_ENV_VARS.md` con tabla del inventario completo de env vars (Production / Preview / Development), required-by, y notes. Patrón ya implementado en Kairos.

### 8.3 Gestión

- **Vercel envs** para apps en Vercel
- **Secret Manager** del proyecto GCP correspondiente para apps en Cloud Run
- **Nunca** secrets en `.env.local` commiteado
- **Nunca** secrets en `~/.zshrc` (chmod 644 — world-readable). Usar `~/.atlax-ai/<project>.env` con `chmod 600`
- **Rotación**: cada secret tiene un owner y una fecha de rotación documentada

## 9. Invariantes transversales (no negociables)

Estos invariantes aplican a **toda app** de la suite, sea cual sea el stack:

### 9.1 Seguridad

- Auth-first: todo router empieza con auth middleware como primera línea
- `getAuthContext()` (o equivalente) en TODA route handler — incluso endpoints "públicos" como taxonomía
- No `NODE_ENV === "production"` como gate de auth — usar env var explícita `<FEATURE>_AUTH_DISABLED`
- CORS allowlist explícita, comparar contra `new URL(origin).origin`
- CSP obligatorio en apps web

### 9.2 Datos

- Multi-tenant by default: toda tabla con `workspace_id` o equivalente, RLS activado en Postgres
- Transacciones obligatorias cuando una operación escribe en 2+ tablas
- State machines: `UPDATE ... WHERE status='expected' RETURNING *`, NUNCA read-check-write
- Bulk INSERT en loops: `.values([...])` único, no `for-await-insert`
- Migraciones con `.down.sql` en el mismo commit

### 9.3 Observabilidad

- Correlation ID `X-Request-ID` propagado E2E (request → handler → outbound calls)
- Logs estructurados (JSON) — nunca `console.*` en servidor de producción
- Health endpoint canónico `GET /api/health` retornando `{ status, db, timestamp }`

### 9.4 Resiliencia

- Todo `fetch` outbound con `AbortSignal.timeout()` — default 15s internas, 30s terceros
- Retry con exponential backoff + jitter
- In-memory caches con TTL bounded

### 9.5 Code quality

- Bun para todo (test, run, install)
- Strict Zod en update schemas (`.strict()` + `.refine(o => Object.keys(o).length > 0)`)
- Constants en `@<app>/types`, nunca strings literales repetidas en 2+ archivos
- Tests co-located en `__tests__/`

### 9.6 Email transaccional interno

Para apps cuyos destinatarios son `@atlax360.com` (apps internas Atlax) — patrón canónico:

- **Gmail API + Service Account + Domain-Wide Delegation**, autenticado vía **Vercel OIDC → Workload Identity Federation → SA Impersonation**. Cero credenciales long-lived (ver ADR-0011).
- **Buzón funcional dedicado por app**: `<app>-bot@atlax360.com` (`kairos-bot@`, `harvest-bot@`, etc.).
- **Service Account por app**: `<app>-mailer@atlax-ai-pro.iam.gserviceaccount.com`.
- **Scope único** `https://www.googleapis.com/auth/gmail.send`. Sin scopes adicionales.
- **DKIM/SPF/DMARC heredados** del dominio `atlax360.com` ya autenticado en Workspace.
- **NO SendGrid / Resend / Postmark** para destinatarios internos: es sobre-arquitectura (paga ~$240/año + DPA + 4º vendor para algo que Workspace entrega gratis con mejor auditabilidad).

Cuándo migrar a proveedor transaccional externo: ver ADR-0011 §"When to migrate to a transactional provider".

## 10. Decisiones tomadas (catálogo)

Cada decisión arquitectónica relevante de la suite va con un ID `D-XXX` y un ADR Michael Nygard detallado en [`docs/adr/`](./adr/).

| ID    | Decisión                                                                              | Status                   | Date       | ADR                                                       |
| ----- | ------------------------------------------------------------------------------------- | ------------------------ | ---------- | --------------------------------------------------------- |
| D-001 | Una sola Consent Screen "Atlax 360 AI Suite", N OAuth Clients                         | Proposed                 | 2026-05-09 | [ADR-0001](./adr/0001-consent-screen-y-oauth-clients.md)  |
| D-002 | Subdominios canónicos `<app>.atlax360.ai` para PRO (dominio propio del grupo)         | Accepted                 | 2026-05-09 | [ADR-0002](./adr/0002-subdominios-atlax360-ai.md)         |
| D-003 | Bun como runtime obligatorio                                                          | Accepted (cross-project) | 2026-04    | [ADR-0003](./adr/0003-bun-runtime.md)                     |
| D-004 | Conventional Commits + Squash merge                                                   | Accepted (cross-project) | 2026-04    | [ADR-0004](./adr/0004-conventional-commits-squash.md)     |
| D-005 | Workload Identity Federation GCP↔GitHub                                               | Proposed                 | 2026-05-09 | [ADR-0005](./adr/0005-workload-identity-federation.md)    |
| D-006 | `vercel.ts` sobre `vercel.json` cuando exista config                                  | Proposed                 | 2026-05-09 | [ADR-0006](./adr/0006-vercel-ts-sobre-vercel-json.md)     |
| D-007 | Vercel AI Gateway por defecto en lugar de provider-specific SDKs                      | Proposed                 | 2026-05-09 | [ADR-0007](./adr/0007-vercel-ai-gateway.md)               |
| D-008 | NO Edge Functions, SÍ Fluid Compute                                                   | Accepted                 | 2026-02    | [ADR-0008](./adr/0008-no-edge-functions-fluid-compute.md) |
| D-009 | `atlax360.ai` es el dominio canónico de la suite AI; `atlax.ai` no pertenece al grupo | Accepted (v0.3)          | 2026-05-09 | [ADR-0009](./adr/0009-dominio-atlax360-ai-canonico.md)    |
| D-010 | Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home del doc canónico       | Accepted                 | 2026-05-10 | [ADR-0010](./adr/0010-repo-dedicado-ai-suite-platform.md) |
| D-011 | Email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD (no SendGrid)     | Proposed                 | 2026-05-11 | [ADR-0011](./adr/0011-email-transaccional-interno.md)     |
| D-012 | `@atlax/auth` postponed — cada app mantiene su auth hasta criterios de reactivación   | Proposed                 | 2026-05-11 | [ADR-0012](./adr/0012-atlax-auth-postponed.md)            |

## 11. Estado actual de adopción (snapshot 2026-05-10)

**Leyenda**: ✅ cumple | 🟡 parcial / en progreso | ❌ deuda técnica | ⊘ no aplica por categoría (ver §3.1)

| App                    | Repo                                               | Deploy                                              | Auth                              | Observability                  | OAuth Client                          | Subdominio                  |
| ---------------------- | -------------------------------------------------- | --------------------------------------------------- | --------------------------------- | ------------------------------ | ------------------------------------- | --------------------------- |
| Kairos                 | github.com/joserragonzalez/kairos                  | Vercel (kairos)                                     | Supabase + Google (en config)     | ✅ pino                        | TBD (en curso)                        | kairos.atlax360.ai          |
| atlax-claude-dashboard | (interno)                                          | Cloud Run                                           | NextAuth + Keycloak Axesor        | TBD                            | atlax-claude-dashboard (Axesor realm) | dashboard.atlax360.ai (TBD) |
| atlax-langfuse-bridge  | github.com/Atlax-360-Test-IA/atlax-langfuse-bridge | Cloud Run + GCE (atlax-langfuse-prod, europe-west1) | Langfuse PK/SK (sin OAuth Google) | ✅ stderr JSON → Cloud Logging | ⊘ N/A sin OAuth                       | langfuse.atlax360.ai        |
| atlax-observatorios    | (interno)                                          | TBD                                                 | TBD                               | TBD                            | TBD                                   | TBD                         |
| harvest                | (interno)                                          | Vercel                                              | TBD                               | TBD                            | TBD                                   | TBD                         |

Cada proyecto debe completar/corregir su fila al validar este documento.

## 12. Roadmap de adopción

### Fase 0 — Completada (2026-05-10) ✅

- Doc v0.3 publicado en Kairos (host transitorio)
- Validación langfuse-bridge completada (13/16 invariantes, 0 anti-patrones, 5 BG)
- Decisiones D-001..D-009 catalogadas
- Repo dedicado `atlax-360-ai-suite/ai-suite-platform` creado (D-010, v0.4)

### Fase 1 — Consolidación (próximas 4-6 semanas)

- [ ] Validación de harvest contra v0.4 (cuando termine sprint en curso)
- [ ] Validación de atlax-observatorios contra v0.4
- [ ] Decisión sobre atlax-claude-dashboard (Keycloak vs Google OAuth)
- [ ] Bridge: cerrar BG-01/04/05 según prioridad cuando proceda
- [ ] Crear OAuth Client "Kairos" en GCP (dependencia: decidir proyecto GCP host)

### Fase 2 — Plataforma operacional (Q3 2026)

- [ ] 3 proyectos GCP `atlax-ai-{dev,pre,pro}` creados (si justificado por ≥2 apps web-app)
- [ ] Workload Identity Federation activa
- [ ] Log sink central operativo
- [ ] Consent Screen "Atlax 360 AI Suite" en `atlax-ai-pro`

### Fase 3 — Portal (cuando ≥3 apps con usuarios solapados)

- Construir `apps.atlax360.ai` o `atlax360.ai` como launcher
- Single login con cookie `*.atlax360.ai`
- **NO urgente** — defer hasta tener señal real de demanda

## 13. Anti-patrones a evitar

- **Crear OAuth Client por entorno** (dev/pre/pro) — fragmenta identidad y multiplica secrets sin beneficio real
- **Mezclar plataforma con apps en el mismo proyecto GCP** — acopla ciclos de vida
- **Cada app con su propio Consent Screen** — fragmenta marca, duplica reviews
- **Dependencias hardcoded entre apps** — si app A llama a app B vía URL fija, no hay independencia
- **Usar `atlax.ai` como dominio** — ese dominio no pertenece al grupo; el canónico es `atlax360.ai`
- **Construir el portal antes de tener 3 apps con usuarios solapados** — over-engineering
- **Crear `atlax-ai-{dev,pre,pro}` antes de tener 2+ apps web-app que los usen** — over-engineering
- **Diferentes runtimes (npm + bun + yarn) entre apps de la suite** — fricción de DX para devs que rotan

## 14. Cómo proponer cambios a este documento

1. Crear PR contra [atlax-360-ai-suite/ai-suite-platform](https://github.com/atlax-360-ai-suite/ai-suite-platform) modificando `docs/SPEC.md`
2. Tag `[shared-platform]` en el título de la PR
3. Notificar a owners de los proyectos afectados
4. Decisión vía consenso de owners de las apps en producción

## 15. Glosario

| Término                          | Definición                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Atlax 360 AI Suite**           | Categoría de apps Atlax orientadas a productividad con IA, con identidad y plataforma compartida |
| **Plataforma**                   | Recursos transversales que cambian poco (auth, DNS, observability)                               |
| **App**                          | Aplicación individual con su propio repo, ciclo de vida y deploy                                 |
| **Consent Screen**               | Pantalla de consentimiento OAuth que el usuario ve al loguearse — vive en GCP                    |
| **OAuth Client**                 | Identidad de aplicación dentro de una Consent Screen — uno por app                               |
| **Redirect URI**                 | URL a la que Google envía al usuario tras login — múltiples por client (uno por entorno)         |
| **Workload Identity Federation** | Mecanismo OIDC para que GitHub Actions obtenga credenciales GCP sin Service Account keys         |

---

**Versión**: v0.6 — D-012 `@atlax/auth` postponed (2026-05-11)
**Próxima revisión**: tras feedback de harvest, atlax-claude-dashboard y atlax-observatorios
