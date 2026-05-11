# ADR-0002 · Subdominios canónicos `<app>.atlax360.ai`

- **Status**: Accepted
- **Date**: 2026-05-09
- **Scope**: all
- **Implements**: D-002

## Context

La suite "Atlax 360 AI Suite" tiene múltiples apps que necesitan ser accesibles por URL: Kairos, atlax-claude-dashboard, langfuse, harvest, atlax-observatorios, y futuras.

El grupo Atlax 360 dispone del dominio `atlax360.ai` (registrado en DonDominio) y de `atlax360.com` (uso corporativo general). La pregunta es: ¿qué patrón de URL canónico adoptamos para producción y entornos no productivos?

Fuerzas en juego:

- **Reconocibilidad de marca**: el usuario debe asociar la URL con Atlax 360 inmediatamente.
- **Operación SSO**: si el portal unificador futuro (`apps.atlax360.ai`) usa cookies de dominio `*.atlax360.ai`, todas las apps necesitan vivir bajo ese apex.
- **Coste DNS**: cada subdominio requiere CNAME en DonDominio + cert SSL — mínimo, pero no cero.
- **Onboarding de apps nuevas**: el patrón debe ser predecible (cualquier dev sabe qué URL le toca a su app sin preguntar).

## Decision

**Todas las apps de la suite usan subdominios bajo el apex `atlax360.ai`:**

| Patrón                             | Uso                                          |
| ---------------------------------- | -------------------------------------------- |
| `<app>.atlax360.ai`                | Producción                                   |
| `<app>.pre.atlax360.ai`            | Pre/staging                                  |
| `<app>.dev.atlax360.ai`            | Dev (opcional — Vercel preview suele bastar) |
| `apps.atlax360.ai` o `atlax360.ai` | Portal unificador (futuro, Capa 5)           |

Donde `<app>` es el nombre canónico (`kairos`, `dashboard`, `langfuse`, `harvest`, `observatorios`). Sin guiones largos ni acrónimos crípticos.

DNS gestionado en DonDominio. SSL automático vía Let's Encrypt (Vercel) o Cloud Run managed certs.

## Alternatives Considered

### Alternativa A — Subdominios bajo `atlax.ai`

Como inicialmente parecía intuitivo (más corto, sin el sufijo `360`).

- **Pro**: URLs más cortas.
- **Contra**: el dominio `atlax.ai` **no pertenece al grupo** — está registrado por un tercero. Usarlo internamente induciría confusión, vendor-lock externo y riesgo de spoofing.
- **Descartada formalmente en D-009 / ADR-0009**.

### Alternativa B — Subdominios bajo `atlax360.com`

Reutilizar el dominio corporativo general.

- **Pro**: dominio ya en uso para email Atlax 360 (`@atlax360.com`).
- **Contra**: `atlax360.com` se usa para web corporativa y operación general; mezclarlo con la suite AI genera ambigüedad de marca y exposición de la web corporativa al perímetro de la suite AI. La suite AI tiene su propia narrativa ("Atlax 360 AI Suite") y merece dominio propio.
- **Descartada**: separación clara entre web corp (`.com`) y suite AI (`.ai`).

### Alternativa C — Cada app con su propio dominio raíz

`kairos.io`, `harvest-by-atlax.com`, etc.

- **Pro**: máxima independencia de marca por app.
- **Contra**: SSO cross-app imposible sin federación compleja; renueva N dominios; rompe coherencia de marca; cada app paga su propio cert + renovación.
- **Descartada**: anti-patrón documentado en SPEC §13.

### Alternativa D — Subdominios profundos por entorno

`pro.kairos.atlax360.ai`, `pre.kairos.atlax360.ai`, etc.

- **Pro**: agrupa por app.
- **Contra**: el patrón con `pro` infijo (`kairos.pro.atlax360.ai`) o sufijo (`pro.kairos.atlax360.ai`) es menos intuitivo; cookies de `*.atlax360.ai` siguen funcionando, pero los certs wildcard necesitan ser `*.kairos.atlax360.ai` por separado.
- **Descartada**: la forma adoptada (`<app>.atlax360.ai` para PRO, `<app>.<env>.atlax360.ai` para no-PRO) es más legible y mantiene el patrón "producción es el caso por defecto, sin sufijo".

## Consequences

### Positivas

- **Marca consistente**: cualquier URL de la suite contiene `atlax360.ai`, reforzando identidad.
- **SSO cross-app viable**: una cookie `*.atlax360.ai` válida para todas las apps cuando se implemente el portal (Capa 5).
- **Patrón predecible**: nuevo dev sabe sin preguntar cuál es la URL de su app.
- **DNS unificado**: una sola zona en DonDominio, un solo proceso de renovación.

### Negativas

- **Dependencia única**: si la zona DNS o el registrar fallan, todas las apps caen. Mitigación: usar registrar con SLA serio + backup del config DNS.
- **Subdominios largos en DEV**: `kairos.dev.atlax360.ai` es verboso. Mitigación: usar Vercel preview URLs como atajo en DEV cuando no se requiera el subdominio formal.

### Neutras

- Cada nueva app añade un CNAME. Operación trivial pero documentada en runbook por app.

## References

- [SPEC.md §4.3 — DNS](../SPEC.md#43-dns)
- [SPEC.md §6.4 — Subdominios por app](../SPEC.md#64-subdominios)
- [ADR-0009 — Dominio canónico `atlax360.ai`](./0009-dominio-atlax360-ai-canonico.md)
- [ADR-0001 — Una Consent Screen, N OAuth Clients](./0001-consent-screen-y-oauth-clients.md) — los redirect URIs siguen este patrón
