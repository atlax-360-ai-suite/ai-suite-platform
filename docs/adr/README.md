# ADRs — Atlax 360 AI Suite

Architectural Decision Records (formato [Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)) formalizando las decisiones de la suite. Cada ADR corresponde a una entrada `D-NNN` del catálogo en [`docs/SPEC.md` §10](../SPEC.md#10-decisiones-tomadas-catálogo).

## Índice

| ADR                                               | Título                                                         | Status                   | Implements | Date       |
| ------------------------------------------------- | -------------------------------------------------------------- | ------------------------ | ---------- | ---------- |
| [0001](./0001-consent-screen-y-oauth-clients.md)  | Una Consent Screen, N OAuth Clients                            | Proposed                 | D-001      | 2026-05-09 |
| [0002](./0002-subdominios-atlax360-ai.md)         | Subdominios canónicos `<app>.atlax360.ai` para PRO             | Accepted                 | D-002      | 2026-05-09 |
| [0003](./0003-bun-runtime.md)                     | Bun como runtime obligatorio                                   | Accepted (cross-project) | D-003      | 2026-04    |
| [0004](./0004-conventional-commits-squash.md)     | Conventional Commits + Squash merge                            | Accepted (cross-project) | D-004      | 2026-04    |
| [0005](./0005-workload-identity-federation.md)    | Workload Identity Federation GCP↔GitHub                        | Proposed                 | D-005      | 2026-05-09 |
| [0006](./0006-vercel-ts-sobre-vercel-json.md)     | `vercel.ts` sobre `vercel.json` cuando exista config           | Proposed                 | D-006      | 2026-05-09 |
| [0007](./0007-vercel-ai-gateway.md)               | Vercel AI Gateway por defecto                                  | Proposed                 | D-007      | 2026-05-09 |
| [0008](./0008-no-edge-functions-fluid-compute.md) | NO Edge Functions, SÍ Fluid Compute                            | Accepted                 | D-008      | 2026-02    |
| [0009](./0009-dominio-atlax360-ai-canonico.md)    | `atlax360.ai` como dominio canónico (no `atlax.ai`)            | Accepted                 | D-009      | 2026-05-09 |
| [0010](./0010-repo-dedicado-ai-suite-platform.md) | Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home | Accepted                 | D-010      | 2026-05-10 |

## Formato canónico (Michael Nygard)

```markdown
# ADR-NNNN · Título

- **Status**: Proposed | Accepted | Superseded | Deprecated
- **Date**: YYYY-MM-DD
- **Scope**: all
- **Implements**: D-NNN
- **Supersedes**: ADR-MMMM (si aplica)

## Context

[Por qué surge esta decisión, qué problema resuelve, qué fuerzas operan]

## Decision

[Qué se decide, formulado como afirmación clara]

## Alternatives Considered

[Opciones evaluadas y por qué se descartaron]

## Consequences

[Positivas, negativas, neutras. Lo que cambia tras adoptar la decisión]

## References

[SPEC.md §X, HANDOFFs, validation reports, otros ADRs]
```

## Reglas

- **Inmutables**: los ADRs no se editan tras ser `Accepted`. Si una decisión cambia, crear ADR nuevo con `Supersedes: ADR-MMMM` y marcar el viejo como `Superseded`.
- **Numeración**: 4 dígitos (`0001`, `0010`, `0100`). Nunca se reutilizan números aunque un ADR se borre durante un PR sin mergear.
- **Vínculo con SPEC §10**: bidireccional — el SPEC referencia el ADR, el ADR referencia la entrada D-NNN.
- **Scope tagging obligatorio** (regla CLAUDE.md global): cada ADR lleva `Scope: all | applicable | <project>`.

## Cómo proponer un ADR nuevo

1. Numerar como el siguiente disponible (mirar último archivo en este directorio).
2. Asignar `Status: Proposed` inicialmente.
3. PR con tag `[shared-platform]` siguiendo `CONTRIBUTING.md`.
4. Tras consenso de owners, cambiar `Status: Accepted` en el mismo PR o uno posterior.
5. Si formaliza una D-NNN ya catalogada en SPEC, no hace falta nueva D — solo añadir link en §10.
