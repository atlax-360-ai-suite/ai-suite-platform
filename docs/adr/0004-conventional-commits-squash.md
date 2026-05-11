# ADR-0004 · Conventional Commits + Squash merge

- **Status**: Accepted (cross-project)
- **Date**: 2026-04
- **Scope**: all
- **Implements**: D-004

## Context

Los proyectos Atlax operan en modo "centaur" (human + AI pair) con un único maintainer activo (Joserra) y crecimiento esperado a equipo distribuido. La historia git debe ser:

- **Legible**: para entender qué cambió y por qué meses después.
- **Lineal**: para que `git bisect` funcione y `git log --graph` sea comprensible.
- **Auditable**: para CHANGELOG automático y para review en post-mortem.

Existen varias convenciones para mensajes de commit y estrategias de merge. La decisión es importante porque cambiarla a posteriori implica reescribir historia o aceptar drift permanente.

## Decision

**Adoptar [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) estricto + Squash merge desde feature branches a `main`.**

### Conventional Commits

Tipos permitidos:

| Tipo       | Uso                                                |
| ---------- | -------------------------------------------------- |
| `feat`     | Nueva funcionalidad                                |
| `fix`      | Bug fix                                            |
| `refactor` | Reestructuración sin cambio funcional              |
| `audit`    | Auditoría / validación / report                    |
| `test`     | Cambios solo en tests (añadir, refactorizar)       |
| `docs`     | Cambios solo en documentación                      |
| `improve`  | Mejora incremental (no es fix ni feature completo) |
| `ops`      | Operaciones de infraestructura, CI, deploy         |
| `chore`    | Tooling, dependencies, configuración menor         |

Formato: `<type>(<scope>): <description>`

Ejemplos válidos:

- `feat(governance): baseline LICENSE + CHANGELOG`
- `fix(reconciler): detectar cobertura parcial del bridge`
- `docs(spec): añadir D-011 sobre @atlax/auth [shared-platform]`

### Squash merge

- Todas las feature branches mergean a `main` con **squash + delete branch**.
- El squash commit usa el título de la PR (típicamente el mismo del primer commit de la rama).
- Tags semver crean los puntos formales de release (no los commits individuales).

### Branch naming

- `feat/<description>` — nueva funcionalidad
- `fix/<description>` — bug fix
- `refactor/<description>` — reestructuración
- `audit/<description>` — auditoría
- `test/<description>` — tests
- `docs/<description>` o `docs/spec-vX.Y` — docs (en repo SPEC, el segundo patrón)
- `improve/<description>` — mejora
- `ops/<description>` — operaciones
- `chore/<description>` — tooling

## Alternatives Considered

### Alternativa A — Merge commits sin squash

Preservar la historia completa de la rama en `main`.

- **Pro**: máxima trazabilidad de commits individuales.
- **Contra**: `git log` se llena de commits "WIP", "fix typo", "address review". Hace `git bisect` más lento e `git log --oneline` ilegible.
- **Descartada**: la trazabilidad fina ya vive en la PR (review history, commits originales si se necesitan).

### Alternativa B — Rebase merge

Aplicar los commits de la rama uno a uno sobre `main`.

- **Pro**: historia lineal y commits preservados.
- **Contra**: cada commit individual debe pasar CI, lo que es costoso. Y los "fix typo" siguen apareciendo en `git log`.
- **Descartada**: el coste excede el beneficio dado que el equipo es pequeño y la PR ya es la unidad de discusión.

### Alternativa C — No usar Conventional Commits, solo mensajes libres

Mayor libertad para el autor.

- **Pro**: cero fricción para autores nuevos.
- **Contra**: imposibilita CHANGELOG automático (`standard-version`, `release-please`); dificulta filtrado por tipo (`git log --grep="^feat"`); reduce coherencia narrativa cross-project.
- **Descartada**: el coste de aprendizaje del estándar (15 min para un dev nuevo) es trivial frente al beneficio operacional.

### Alternativa D — Conventional Commits + merge commits (sin squash)

Combinación intermedia.

- **Pro**: estandariza el formato pero preserva la historia.
- **Contra**: los commits intermedios "WIP" siguen en `main`. La carga de mantener TODOS los commits conforme al estándar (no solo el de squash) es mayor.
- **Descartada**: si el squash commit cumple el estándar, los intermedios pueden ser libres (más amigable para iteración).

## Consequences

### Positivas

- **Historia lineal**: `git log --oneline` muestra solo squash commits, fácil de auditar.
- **CHANGELOG automatizable**: tooling como `release-please` genera entradas a partir de los tipos.
- **`git bisect` rápido**: cada commit en `main` es una unidad funcional completa.
- **PRs son la unidad de discusión**: los commits intermedios viven en GitHub PR, no contaminan `main`.
- **Branch naming homogéneo**: facilita filtros y automatización.

### Negativas

- **Pérdida de granularidad** en historia merged: si una rama tenía 12 commits, en `main` solo aparece 1. Mitigación: GitHub preserva los commits originales en la PR.
- **Curva de aprendizaje** del estándar Conventional Commits: 15 min de lectura.

### Neutras

- Las feature branches pueden tener commits "WIP", "fix typo" — solo el squash commit final debe cumplir el estándar.

## References

- [SPEC.md §7.2 — Ciclo Local → DEV → PRE → PRO](../SPEC.md#72-ciclo-local--dev--pre--pro)
- [SPEC.md §7.3 — Branch protection](../SPEC.md#73-branch-protection)
- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- CLAUDE.md global del usuario (regla "Branch Protection — protocolo de rescate")
