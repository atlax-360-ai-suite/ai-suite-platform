# ADR-0011 · Email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD

- **Status**: Proposed
- **Date**: 2026-05-11
- **Scope**: all (aplica a cualquier app interna de la suite que envíe email a destinatarios `@atlax360.com`)
- **Implements**: D-011

## Context

Las apps de la suite necesitan enviar emails transaccionales: notificaciones HITL (aprobaciones pendientes), alertas operativas, recordatorios. Para apps **internas** (todos los destinatarios bajo el dominio `atlax360.com` gestionado por Google Workspace) hay varios caminos:

1. **nodemailer + SMTP Gmail con App Password** — patrón legacy. Google marca App Passwords como deprecadas y la política Atlax 360 las veta. Inicialmente adoptado en Kairos como TD-EMAIL pero bloqueado en producción por falta de habilitación.
2. **SendGrid u otro proveedor transaccional externo** — patrón usado en Orvian (`@orvian/email`, dominio `alerts.yndika.com`). Justificado en Orvian porque envía a deudores externos (necesita IP reputation, suppression list, bounce handling cross-domain). **Sobre-arquitectura para apps internas**: paga ~$720/3 años + DPA + cuarto vendor para algo que Workspace ya entrega gratis.
3. **Gmail API directa con OAuth2 cliente humano** — requiere refresh token gestionado por una persona, no apta para cuenta robot.
4. **Gmail API con Service Account + Domain-Wide Delegation, autenticando via Vercel OIDC + Workload Identity Federation** — patrón Google-nativo recomendado en 2026 para apps server-side que envían en nombre de un buzón funcional. Cero credenciales de larga duración. Alineado con ADR-0005 (WIF GCP↔GitHub) extendido al runtime (Vercel↔GCP).

El caso Atlax 360 / Kairos:

- Todos los destinatarios son `@atlax360.com` (empleados del grupo).
- Existe Google Workspace pagado con DKIM/SPF/DMARC ya configurados sobre `atlax360.com`.
- Existe infraestructura GCP (carpeta `atlax-ai`, proyectos siguiendo §4.1 del SPEC).
- Volumen bajo (cientos de emails/día por app).
- Sin requisitos de deliverability cross-domain.

## Decision

**Adoptar Gmail API + Service Account + Domain-Wide Delegation, autenticado vía Vercel OIDC → Workload Identity Federation → Service Account Impersonation, como patrón canónico para email transaccional interno en apps Atlax 360 AI Suite.**

Componentes fijos:

- **Buzón funcional dedicado por app** en Google Workspace, naming convención `<app>-bot@atlax360.com` (ej. `kairos-bot@atlax360.com`).
- **Service Account dedicado** en el proyecto GCP `atlax-ai-pro`, naming `<app>-mailer@atlax-ai-pro.iam.gserviceaccount.com`. SIN claves JSON.
- **Workload Identity Pool** llamado `vercel` en el proyecto GCP, un **Provider OIDC por app** con condición `assertion.project_id == '<vercel-project-id>' && assertion.environment == 'production'`.
- **DWD authorisation** en Google Workspace Admin (`admin.google.com/ac/owl/domainwidedelegation`) con scope **único** `https://www.googleapis.com/auth/gmail.send`. Sin scopes adicionales.

Flujo de autenticación en runtime (cada llamada):

1. Vercel Fluid Compute genera un JWT OIDC firmado por `oidc.vercel.com/<team-slug>` (45 min TTL).
2. STS de GCP intercambia ese JWT por un token federado, validando las condiciones del Provider.
3. IAM Credentials API impersona el Service Account (`*-mailer`), devolviendo un access token con scope `cloud-platform`.
4. La app llama `iamcredentials.projects.serviceAccounts.signJwt` para firmar un JWT con `sub: <buzón-funcional>@atlax360.com` y scope `gmail.send`.
5. Ese JWT se intercambia vía `urn:ietf:params:oauth:grant-type:jwt-bearer` por un access_token Gmail autorizado a enviar AS ese buzón.
6. Gmail API `users.messages.send` envía el correo.

El SDK de referencia es `googleapis` + `google-auth-library` + `@vercel/oidc`. El cliente se publica como `@kairos/email` en Kairos como **reference implementation**. Cuando una segunda app de la suite necesite email transaccional, se extrae el cliente a `@atlax/email` (repo dedicado, fuera de `ai-suite-platform` que es solo specs por CLAUDE.md). No promovemos a package compartido antes de tener el segundo caso real (principio D-007 — defer cuando puedas).

## Alternatives Considered

### Alternativa A — SendGrid / Postmark / Resend (proveedor transaccional externo)

- **Pro**: panel admin, suppression list, IP warm-up automático, bounce webhooks, métricas listas.
- **Contra**: añade un vendor externo al blast radius, $20-30/mes/app sin escala, DPA y SCCs extra (Schrems II para Resend US, EU residency parcial), 2 hops adicionales (Vercel → SendGrid → Google) cuando el destino siempre fue Google. Domain Authentication separada (subdominio `notifications.atlax360.com` con DKIM/SPF/DMARC propios) cuando `atlax360.com` ya está autenticado por Workspace.
- **Descartada para apps internas**. **Aceptable cuando**: una app empieza a enviar a destinatarios externos al dominio `atlax360.com`. Ver §6 sobre migración.

### Alternativa B — SMTP relay Workspace + XOAUTH2

- **Pro**: protocolo SMTP estándar, integración fácil con `nodemailer`.
- **Contra**: XOAUTH2 sobre SMTP relay requiere igualmente DWD (mismo coste) pero añade complejidad de transporte (STARTTLS, timeouts, manejo de conexión persistente). Sin observabilidad nativa equivalente a Gmail API (Sent folder, Cloud Logging).
- **Descartada**: misma autenticación con peor experiencia operativa.

### Alternativa C — App Passwords + nodemailer

- **Pro**: setup de 10 minutos.
- **Contra**: Google los marca como legacy ("menos seguras que estándares modernos"). Credencial long-lived en env var. Política de seguridad Atlax los veta. Si el super-admin los habilita, queda como deuda de seguridad permanente.
- **Descartada**: bloqueada por política Atlax.

### Alternativa D — Service Account con JSON key pegada en Vercel env

- **Pro**: alternativa "fácil" a WIF.
- **Contra**: clave de larga duración rotable manualmente. Google lo desaconseja activamente desde 2024 (mismo argumento que ADR-0005).
- **Descartada**: incompatible con security baseline 2026.

## Consequences

### Positivas

- **Cero credenciales long-lived** en runtime (Vercel) y CI/CD (GitHub, ya cubierto por ADR-0005).
- **Coste $0**: Gmail API es ilimitada bajo cuota Workspace (project quota 80M units/día, `messages.send` = 100 units, ~800k emails/día project; 2k destinatarios externos/día por buzón funcional, irrelevante para envío interno).
- **Auditabilidad nativa**: cada email enviado aparece en el folder "Sent" del buzón funcional + Cloud Logging del SA + Workspace Audit logs. Tres fuentes de verdad.
- **DKIM/SPF/DMARC heredados**: el dominio `atlax360.com` ya está autenticado en Workspace; los emails enviados pasan los chequeos sin configuración adicional.
- **Alineamiento con ADR-0005**: WIF ya estaba decidido para CI/CD; este ADR lo extiende al runtime con el mismo patrón conceptual.
- **Principio de mínimo privilegio**: scope único `gmail.send`. El SA no puede leer ni modificar emails.

### Negativas

- **Setup inicial mayor que un API key SendGrid** (~45 min: SA + WIF Pool + Provider + DWD authorization + Vercel env vars). Mitigado: solo se hace una vez por app, documentado en este ADR.
- **Curva de aprendizaje**: el flujo de 3 leg (WIF → SA → DWD JWT) es más complejo de entender que "API key + POST". Mitigado: el cliente `@kairos/email` lo encapsula completamente; el caller solo ve `sendEmail({to, subject, html})`.
- **Acoplamiento Vercel↔GCP**: si la app migra a Cloud Run u otro hosting, hay que rehacer la federación. Mitigado: el cliente está abstracto detrás de `sendEmail` — solo cambia `gmail-client.ts`, no la lógica de negocio.
- **Quota de 2k destinatarios/día por buzón funcional**: aplica solo a envíos a destinatarios externos al Workspace. Para apps con destinatarios mixtos (algunos `@atlax360.com`, algunos externos), reevaluar contra Alternativa A.

### Neutras

- El buzón funcional `<app>-bot@atlax360.com` consume 1 licencia Workspace por app. Coste marginal aceptado.
- DWD requiere autorización manual por el super-admin Workspace. Mitigado: documentado paso a paso en el ADR; se hace una vez por app.

## When to migrate to a transactional provider (Alternativa A)

Reconsiderar SendGrid u otro cuando se cumpla cualquiera de:

1. **La app empieza a enviar a destinatarios externos** (no `@atlax360.com`) de forma sistemática.
2. **Volumen supera 1.500 emails/día sostenido** a destinatarios externos al Workspace, acercándose a la quota de 2k del buzón funcional.
3. **Se necesita suppression list global** entre apps (un usuario que se da de baja en Kairos también deja de recibir Harvest).
4. **Reportes regulados** (DMARC quarantine reports, bounce metrics agregados con SLA) que Workspace no expone con la granularidad que el negocio necesita.

El cliente `@kairos/email` está diseñado para que la migración sea cambiar `gmail-client.ts` por un `sendgrid-client.ts` que implemente el mismo contrato `sendEmail(opts): Promise<SendEmailResult>`. Estimado: <1 día de trabajo + DNS de Domain Authentication.

## References

- [SPEC.md §4 — Capa 1 Plataforma compartida](../SPEC.md#4-capa-1--plataforma-compartida) — define los proyectos GCP `atlax-ai-{dev,pre,pro}` donde vive el SA
- [ADR-0005 — Workload Identity Federation GCP↔GitHub](./0005-workload-identity-federation.md) — patrón análogo extendido a runtime
- [ADR-0008 — NO Edge Functions, SÍ Fluid Compute](./0008-no-edge-functions-fluid-compute.md) — Vercel Fluid Compute soporta el SDK `googleapis` y el flujo OIDC
- [Vercel OIDC docs](https://vercel.com/docs/oidc) — issuer URL por team, audience, TTL 45min
- [Vercel → GCP via OIDC](https://vercel.com/docs/oidc/gcp) — ejemplo canónico de `ExternalAccountClient.fromJSON`
- [Google Workspace Admin — DWD best practices](https://support.google.com/a/answer/14437356) — DWD aceptable con scope mínimo y buzón funcional dedicado (caso de uso de este ADR)
- [Gmail API quotas](https://developers.google.com/workspace/gmail/api/reference/quota) — 80M units/día project, `messages.send` = 100 units
- Implementación de referencia: `kairos/packages/email/` (commit pendiente)
