# ADR-0009 · `atlax360.ai` como dominio canónico de la suite (no `atlax.ai`)

- **Status**: Accepted
- **Date**: 2026-05-09
- **Scope**: all
- **Implements**: D-009

## Context

Durante el draft inicial del SPEC (v0.1 y v0.2 en Kairos como host transitorio), aparecieron referencias inconsistentes al dominio de la suite: `atlax.ai`, `atlax-ai.com`, `atlax360.ai`. Una revisión cuidadosa de los dominios reales que el grupo Atlax 360 posee reveló:

- **`atlax360.com`** — dominio corporativo principal del grupo Atlax 360. Web pública, email `@atlax360.com`.
- **`atlax360.ai`** — dominio adquirido específicamente para la suite AI. Registrado en DonDominio.
- **`atlax.ai`** — **NO pertenece al grupo**. Registrado por un tercero. Cualquier referencia interna que asuma propiedad de `atlax.ai` es incorrecta y peligrosa.

Riesgo identificado: si código o configs apuntan a `<algo>.atlax.ai` confiando en propiedad, exponemos a la suite a:

- **Spoofing**: un atacante con control de `atlax.ai` puede crear `<phishing>.atlax.ai` indistinguible de la suite legítima.
- **Pérdida de control**: si el OAuth Consent Screen autoriza `atlax.ai`, Google verifica que el dominio está bajo nuestro control — fallaría y nos abriría la puerta a takeover.
- **Confusión de usuarios**: emails internos referenciando `*.atlax.ai` llevan a sitios externos.

## Decision

**`atlax360.ai` es el dominio canónico de la suite Atlax 360 AI Suite. `atlax360.com` es el dominio corporativo del grupo. Cualquier referencia a `atlax.ai` en código, docs, configs o emails es un bug.**

Aplicaciones concretas:

- Subdominios canónicos: `<app>.atlax360.ai` (ver ADR-0002).
- Emails de servicio: `<service>@atlax360.com` (ej. `kairos@atlax360.com`).
- Authorized domains en Google OAuth Consent: `atlax360.ai`, `atlax360.com`, `supabase.co`. **Nunca** `atlax.ai`.
- URLs en docs, ADRs, READMEs: solo `atlax360.ai` y `atlax360.com`.

**Test de cumplimiento** (verificado en `atlax-langfuse-bridge/tests/naming-convention.test.ts`):

- Regex `\b[a-z][a-z0-9-]*\.atlax\.ai\b` no debe matchear ninguna línea bajo control del repo.
- Excepciones legítimas (referencias a filenames legacy como `atlax-ai-shared-platform.md`) se sanitizan explícitamente antes del match.

## Alternatives Considered

### Alternativa A — Usar `atlax.ai` y adquirirlo

Comprar el dominio para evitar la situación.

- **Pro**: URLs más cortas, marca más directa.
- **Contra**: el actual propietario puede no querer venderlo o pedir un precio prohibitivo. Aunque se comprara, ya hay deuda histórica en cómo el grupo se identifica (`@atlax360.com` para emails, `atlax360.com` para web). Cambiar todo añade churn.
- **Descartada**: el coste de adquisición + migración excede el beneficio.

### Alternativa B — Mantener ambiguo, "cualquiera que parezca atlax"

Tolerar que algunas configs usen `atlax.ai` y otras `atlax360.ai`.

- **Pro**: cero esfuerzo a corto plazo.
- **Contra**: drift garantizado, riesgo de seguridad (spoofing), confusión cross-team. El primer phishing real podría golpear la suite.
- **Descartada**: defensa en profundidad — un solo dominio canónico, comprobado con test.

### Alternativa C — Usar `atlax360.com` para todo (web + suite)

Eliminar `atlax360.ai` del mapa.

- **Pro**: una sola zona DNS, una sola identidad.
- **Contra**: mezclar la suite AI con la web corporativa diluye la narrativa "Atlax 360 AI Suite" y expone la web pública al perímetro de la suite. La separación `.com` vs `.ai` aporta claridad y compartimentalización.
- **Descartada**: la separación es valor real.

## Consequences

### Positivas

- **Cero ambigüedad de identidad**: una sola URL canónica.
- **Protección contra spoofing**: el atacante con `atlax.ai` no puede impersonar la suite.
- **OAuth Consent Screen consistente**: solo dominios autorizados que controlamos.
- **Test enforced**: el regex en `naming-convention.test.ts` bloquea drift en CI.

### Negativas

- **URL ligeramente más larga** (`kairos.atlax360.ai` vs el ficticio `kairos.atlax.ai`). Aceptable.
- **Coste continuo de la zona DNS** en DonDominio. Trivial.

### Neutras

- Devs nuevos deben aprender que `atlax.ai` es prohibido. Documentado en CLAUDE.md global, CONTRIBUTING del SPEC, y enforced por test.

## References

- [SPEC.md §4.3 — DNS](../SPEC.md#43-dns)
- [SPEC.md §13 — Anti-patrones ("Usar `atlax.ai` como dominio")](../SPEC.md#13-anti-patrones-a-evitar)
- [ADR-0001 — Una Consent Screen, N OAuth Clients](./0001-consent-screen-y-oauth-clients.md) — Authorized domains
- [ADR-0002 — Subdominios `<app>.atlax360.ai`](./0002-subdominios-atlax360-ai.md)
- `~/work/atlax-langfuse-bridge/tests/naming-convention.test.ts` — test enforced
