# ADR-0001 · Una sola Consent Screen "Atlax 360 AI Suite" con N OAuth Clients

- **Status**: Proposed
- **Date**: 2026-05-09
- **Scope**: all
- **Implements**: D-001

## Context

La categoría "Atlax 360 AI Suite" va a tener múltiples apps (Kairos, atlax-claude-dashboard, harvest, atlax-langfuse-bridge, atlax-observatorios, futuras) que en su mayoría requieren autenticación de usuario vía Google OAuth — bien para SSO interno, bien para acceder a scopes específicos de Google APIs (Drive, Calendar, Gmail).

Google Cloud Platform organiza esto en dos artefactos:

- **OAuth Consent Screen** — la pantalla de consentimiento que el usuario ve al loguearse por primera vez. Marca, política de privacidad, dominios autorizados, lista de scopes.
- **OAuth Client** — identidad de aplicación dentro de una Consent Screen. Tiene client_id, client_secret y una lista de redirect URIs.

Cada Consent Screen puede tener N OAuth Clients. La pregunta arquitectónica es: ¿una Consent Screen para toda la suite y N clients (uno por app), o una Consent Screen por app?

Fuerzas en juego:

- **Marca**: el usuario debe entender que Kairos, harvest y dashboard son parte del mismo grupo (Atlax 360).
- **Aislamiento**: la filtración del secret de una app no debe comprometer a las demás.
- **Verificación de Google**: si una app se comercializa (External + verified), Google hace review manual. Cada Consent Screen separada multiplica las reviews.
- **Mantenimiento legal**: política de privacidad y términos de uso.

## Decision

**Una sola OAuth Consent Screen "Atlax 360 AI Suite" para toda la categoría, con un OAuth Client por aplicación.**

Cada OAuth Client lleva el nombre canónico de su app (`Kairos`, `Dashboard`, `Langfuse-bridge`, `Harvest`, `Observatorios`). Cada uno tiene redirect URIs para **todos los entornos** (`http://localhost:*`, `<app>.dev.atlax360.ai`, `<app>.pre.atlax360.ai`, `<app>.atlax360.ai`) — un solo client cubre el ciclo de vida completo.

La Consent Screen vive en el proyecto GCP `atlax-ai-pro` (Capa 1 del SPEC). Authorized domains: `atlax360.ai`, `atlax360.com`, `supabase.co`. Privacy policy y terms: `https://atlax360.ai/legal/privacy` y `terms`.

Client Secrets viven en **Secret Manager** del proyecto GCP correspondiente, nunca en `.env.local` ni en repos.

## Alternatives Considered

### Alternativa A — Una Consent Screen por app

Cada app es completamente independiente en GCP: su propia Consent Screen, su propio OAuth Client.

- **Pro**: máximo aislamiento; cada app puede ir a External + verified de forma independiente.
- **Contra**: el usuario ve "Kairos", "Harvest", "Dashboard" como apps inconexas. Cada Consent Screen requiere su propia política legal y verificación de dominios. Multiplica reviews de Google cuando se comercialicen apps. Branding inconsistente.
- **Descartada**: el coste operativo y el daño de branding exceden el beneficio marginal de aislamiento (que se logra a nivel de OAuth Client por separado).

### Alternativa B — Un OAuth Client compartido por todas las apps

Una Consent Screen y un único OAuth Client cuyo client_id se reutiliza en todas las apps de la suite.

- **Pro**: menos artefactos que mantener.
- **Contra**: filtración del secret → revocas todas las apps. Auditoría de OAuth no granular (no sabes qué app generó qué token). Cada app necesitaría TODOS los scopes (intersección creciente sin control).
- **Descartada**: el blast radius es inaceptable para una suite que crecerá a 5+ apps.

### Alternativa C — Un OAuth Client por (app × entorno)

Una Consent Screen con N × M clients (cada app tiene un client `dev`, otro `pre`, otro `pro`).

- **Pro**: aislamiento granular por entorno; revocar PRO no afecta DEV.
- **Contra**: 3× más artefactos que mantener. Cada cambio de scope se replica en 3 lugares. Más secrets que rotar. No hay evidencia de que un compromiso de DEV propague a PRO si los secretos están separados.
- **Descartada**: catalogada como anti-patrón en SPEC §13. Es over-engineering — un solo client con redirect URIs por entorno cubre todos los casos.

## Consequences

### Positivas

- **Branding consistente**: el usuario ve "Atlax 360 AI Suite" en `myaccount.google.com → Apps with access`, con sub-apps identificables ("Kairos accedió a Drive el 2026-05-11", etc.).
- **Una sola review de Google** si alguna app pasa a `External + verified`. Aplica también a verificación de dominios.
- **Una sola política legal**: privacy policy y terms únicos a mantener.
- **Aislamiento por OAuth Client**: filtración de secret → revocación afecta solo a esa app.
- **Permisos evolucionables**: cada app pide los scopes que necesita sin afectar al resto.
- **Auditoría granular**: logs de OAuth segmentados por app.

### Negativas

- **Dependencia compartida**: la Consent Screen vive en `atlax-ai-pro`. Si ese proyecto GCP se rompe, todas las apps tienen problemas de auth. Mitigación: WIF + monitoreo del proyecto + backups del config (`gcloud iap settings export`).
- **Coordinación al añadir scopes nuevos**: si una app añade un scope sensible (ej. `drive.readonly`), se afecta el privacy policy global de la suite.

### Neutras

- Cada app necesita gestionar su propio client_secret. Patrón ya estándar en la industria.

## References

- [SPEC.md §4.2 — OAuth Consent Screen](../SPEC.md#42-oauth-consent-screen)
- [SPEC.md §5 — Capa 2 OAuth Clients](../SPEC.md#5-capa-2--oauth-clients)
- [SPEC.md §13 — Anti-patrones (cita "Crear OAuth Client por entorno")](../SPEC.md#13-anti-patrones-a-evitar)
- [ADR-0002 — Subdominios `<app>.atlax360.ai`](./0002-subdominios-atlax360-ai.md) — define los redirect URIs canónicos
- [ADR-0009 — Dominio canónico `atlax360.ai`](./0009-dominio-atlax360-ai-canonico.md) — define los authorized domains
