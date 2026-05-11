# ADR-0010 · Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home del SPEC

- **Status**: Accepted
- **Date**: 2026-05-10
- **Scope**: all
- **Implements**: D-010

## Context

El documento canónico "Atlax 360 AI Suite — Shared Platform" tuvo un nacimiento iterativo:

- **2026-05-09**: draft v0.1 generado dentro del repo `kairos` como ejercicio cross-project. Vivía en `~/work/kairos/docs/atlax-ai-shared-platform.md`.
- **2026-05-09**: v0.2 consolidando reporte de validación de `atlax-langfuse-bridge`. Sigue en Kairos como **host transitorio**.
- **2026-05-09**: v0.3 corrigiendo el dominio canónico (D-009). Sigue en Kairos.

Esta ubicación tenía un problema arquitectónico: el SPEC es **cross-project** por naturaleza (governance de ≥5 apps), pero vivía en el repo de UNA de esas apps. Cualquier PR al SPEC requería abrir PR contra Kairos, mezclando governance con desarrollo de la app. Riesgos:

- Maintainers de otras apps necesitan acceso a Kairos.
- Branch protection de Kairos no está optimizada para PRs de governance.
- El CI de Kairos corre tests de su código en cada PR al SPEC (lentitud innecesaria).
- Conceptualmente confuso: ¿por qué la spec de la suite vive bajo una app?

El SPEC §3.1 documentaba explícitamente "Kairos es host transitorio" — el plan siempre fue mover el doc cuando se cumpliera un trigger.

## Trigger condition

**El doc canónico se mueve a un repo dedicado cuando hay ≥2 adopciones formales de la suite.**

Estado a 2026-05-10:

- **Kairos** ✅ — host original, en producción `kairos.atlax360.ai`, OAuth Client en curso.
- **atlax-langfuse-bridge** ✅ — validación completada (13/16 invariantes ✅, 0 anti-patrones, 5 BG documentados), PRO operativo en `langfuse.atlax360.ai` desde 2026-05-10, F5 completada.

Trigger cumplido. Se procede al movimiento.

## Decision

**El SPEC canónico se mueve al repo dedicado `atlax-360-ai-suite/ai-suite-platform` en su filename canónico `docs/SPEC.md`. La versión se bumpea de v0.3 (host transitorio) a v0.4 (repo dedicado).**

Cambios mecánicos:

- Crear org GitHub `atlax-360-ai-suite`.
- Crear repo `atlax-360-ai-suite/ai-suite-platform` (público, governance-only).
- Mover `~/work/kairos/docs/atlax-ai-shared-platform.md` v0.3 → `~/work/ai-suite-platform/docs/SPEC.md` v0.4.
- Eliminar el doc de Kairos.
- Actualizar `docs/00-INDEX.md` de Kairos con puntero externo + nota "host transitorio hasta v0.3".
- Actualizar `atlax-langfuse-bridge/CLAUDE.md` línea 29 + audit doc para apuntar al repo nuevo.

Cambios de proceso:

- PRs al SPEC se abren contra el repo dedicado, no contra Kairos.
- Tag `[shared-platform]` obligatorio en el título.
- Branch protection en `main` del repo dedicado.
- Versionado semántico estricto (`vMAJOR.MINOR.PATCH`).

## Alternatives Considered

### Alternativa A — Mantener el SPEC en Kairos indefinidamente

Status quo.

- **Pro**: cero migración.
- **Contra**: violation del principio "doc cross-project en repo de app única"; futuras apps necesitarán acceso a Kairos solo para proponer cambios al SPEC.
- **Descartada**: el coste futuro de no hacerlo crece linealmente con el número de apps.

### Alternativa B — Mover el SPEC a un repo "atlas/docs" general

Repo central de toda la documentación cross-project del grupo.

- **Pro**: un solo repo para todas las decisiones cross.
- **Contra**: mezcla governance de la suite AI con cualquier otra decisión Atlax (legal, IT general, etc.). El SPEC de la AI Suite tiene sus propios reviewers (owners de apps AI), branch protection específica, ciclo de release propio.
- **Descartada**: el coste de fragmentación operacional excede el beneficio de consolidación.

### Alternativa C — Mover el SPEC a `atlax-observatorios`

Otra opción dentro del ecosistema actual.

- **Pro**: `atlax-observatorios` es el "watcher" del ecosistema técnico.
- **Contra**: confunde governance con observabilidad; tampoco es naturalmente cross-project.
- **Descartada**: la solución correcta es un repo dedicado con propósito único.

### Alternativa D — Crear el repo cuando haya ≥3 adopciones

Diferir más.

- **Pro**: aplica más estrictamente "defer cuando puedas".
- **Contra**: el trigger ≥2 ya marcaba el límite operacional. Esperar a 3 producía drift adicional cuando harvest valide.
- **Descartada**: ≥2 es el balance correcto.

## Consequences

### Positivas

- **Governance limpia**: el SPEC vive en su propio repo, con branch protection y CODEOWNERS apropiados.
- **Independencia operacional**: PRs al SPEC no compiten con desarrollo de apps individuales.
- **Discoverability**: cualquier dev de la suite encuentra el SPEC con `gh repo view atlax-360-ai-suite/ai-suite-platform`.
- **Versionado formal**: tags semver permiten que apps referencien una versión específica del SPEC (`v0.4`, `v0.5`, etc.).
- **Espacio para crecer**: ADRs, HANDOFFs, templates futuros caben en el mismo repo sin contaminar apps.

### Negativas

- **Una org y un repo más que mantener**: trivial, GitHub free tier lo cubre.
- **Migration cost**: ~2h de actualizar referencias en Kairos y bridge. Pagado al crear el repo.
- **Coordinación inicial mayor**: cualquier proyecto que validaba contra el SPEC necesita actualizar su `CLAUDE.md` con la URL nueva.

### Neutras

- El SPEC se publica como repo público — coherente con la filosofía "governance es transparente".

## Migration log

| Fecha      | Acción                                                   |
| ---------- | -------------------------------------------------------- |
| 2026-05-09 | Doc v0.1..v0.3 generado en Kairos (host transitorio)     |
| 2026-05-10 | Org `atlax-360-ai-suite` creada                          |
| 2026-05-10 | Repo `atlax-360-ai-suite/ai-suite-platform` creado       |
| 2026-05-10 | Doc movido + bumped a v0.4 (`docs/SPEC.md`)              |
| 2026-05-10 | Kairos: 3 ficheros eliminados, `00-INDEX.md` actualizado |
| 2026-05-10 | Bridge: `CLAUDE.md` + audit doc actualizan URL           |

## References

- [SPEC.md §10 D-010](../SPEC.md#10-decisiones-tomadas-catálogo)
- [SPEC.md §14 — Cómo proponer cambios](../SPEC.md#14-cómo-proponer-cambios-a-este-documento)
- [CHANGELOG.md v0.4.0](../../CHANGELOG.md#v040--2026-05-10) — entrada de migración
- [Kairos HANDOFF-v2.13.md §Cross-project](https://github.com/joserragonzalez/kairos/blob/main/docs/HANDOFF-v2.13.md) — narrativa de la migración desde el lado Kairos
- [Bridge audit `docs/audits/shared-platform-validation-2026-05-09.md`](https://github.com/atlax360/atlax-langfuse-bridge/blob/main/docs/audits/shared-platform-validation-2026-05-09.md) — validación que disparó el trigger ≥2
