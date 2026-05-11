# Contribuir al SPEC de Atlax 360 AI Suite — Shared Platform

Este repo aloja el **documento canónico de gobernanza** que define el patrón de plataforma compartida para la categoría "Atlax 360 AI Suite". Todas las apps de la categoría se rigen por él.

Este no es un repo de código ejecutable: no hay `bun install`, no hay tests automáticos de funcionalidad. La calidad se garantiza mediante **proceso de revisión**, no mediante CI funcional.

---

## Cuándo proponer un cambio

Los cambios al SPEC vienen normalmente de **uno de estos tres eventos**:

1. **Validación de una app contra el SPEC** — un proyecto Atlax (harvest, dashboard, observatorios, etc.) hace una validación y detecta gap entre el SPEC y la realidad. Algunos hallazgos son `KP-NN` (Knowledge Proposal: cambios al SPEC), otros son `AG-NN` (App Gap: deuda del proyecto, no del SPEC).
2. **Aprendizaje cross-project** — un patrón de seguridad / código / data que ha demostrado valor en 2+ proyectos y merece convertirse en invariante.
3. **Decisión arquitectónica nueva** — cuando aparece una situación nueva que requiere una decisión `D-NNN` formalizada.

**No proponer un cambio** si:

- Es específico de un solo proyecto (vive en su propio `CLAUDE.md`).
- Es una preferencia personal sin evidencia cross-project.
- Es una optimización prematura (recordar: principio "defer cuando puedas").

---

## Proceso de PR

### 1. Crear rama

```bash
git checkout -b docs/spec-vX.Y       # para cambios al SPEC
git checkout -b feat/<topic>          # para gobernanza del repo (CONTRIBUTING, ADRs, CI, etc.)
git checkout -b chore/<topic>         # para tooling y fixes menores
```

### 2. Editar

- **Cambios al SPEC**: editar `docs/SPEC.md`, bumpear versión (§header + footer + entrada en `CHANGELOG.md`).
- **ADRs**: crear `docs/adr/NNNN-titulo-kebab.md` siguiendo el formato Michael Nygard. Si formaliza una decisión `D-NNN` del SPEC, enlazar bidireccionalmente.
- **Roadmap**: actualizar `§12` del SPEC + `docs/HANDOFF-current.md`.

### 3. Versionado semántico

- **major** (`v1.0.0`) — reestructura del SPEC o breaking change del patrón
- **minor** (`v0.5.0`) — nueva decisión, nuevo invariante, nueva sección
- **patch** (`v0.4.1`) — correcciones, clarificaciones, snapshot de adopción

Cuando bumpees versión:

- Actualizar header del SPEC (`Status: Accepted vX.Y`)
- Actualizar tabla "Changelog" embebida en el SPEC
- Añadir entrada en `CHANGELOG.md` con `Added` / `Changed` / `Removed` / `Migration notes`
- Actualizar footer del SPEC

### 4. PR

- **Título**: prefijo `docs(spec):` o `docs(adr):` + tag `[shared-platform]`. Ejemplo: `docs(spec): añadir D-011 sobre @atlax/auth [shared-platform]`
- **Cuerpo**: usar PR template — describir motivación, sección modificada, decisión asociada, apps afectadas, owners notificados.
- **Reviewers**: notificar a owners de las apps en producción (ver `CODEOWNERS`).

### 5. Merge

- **Squash merge** obligatorio (mantiene historia lineal en `main`).
- Borrar rama remota tras merge.
- Si el PR introduce nuevo tag (minor/major), crear el tag tras el merge:

```bash
git checkout main && git pull
git tag -a vX.Y.Z -m "vX.Y.Z — <one-line summary>"
git push origin vX.Y.Z
gh release create vX.Y.Z --notes-from-tag
```

---

## Convenciones de redacción

### Markdown

- ATX headings (`##`), nunca subrayado con `===`/`---`
- Listas con `-` (no `*` ni `+`)
- Tablas alineadas (los formateadores de Markdown del editor las ajustan automáticamente)
- Idioma: **español** (el SPEC se redacta en español; ADRs también)
- Código en inline `backticks` para paths, nombres de variables, comandos
- Bloques de código con fence triple + lenguaje (` ```bash`, ` ```typescript`)

### Términos canónicos

| Término correcto                           | Términos a evitar                             |
| ------------------------------------------ | --------------------------------------------- |
| `Atlax 360 AI Suite`                       | `Atlax AI Suite`, `Atlax-AI`                  |
| `atlax360.ai`                              | `atlax.ai` (D-009: no pertenece al grupo)     |
| `atlax-360-ai-suite/<repo>`                | `atlax-ai/<repo>`, `atlax360-ai-suite/<repo>` |
| `web-app` / `edge-tooling` / `server-only` | "aplicación web", "CLI", etc.                 |

### Tablas de adopción (§11)

Leyenda canónica:

- ✅ cumple
- 🟡 parcial / en progreso
- ❌ deuda técnica
- ⬜ no validado todavía
- ⊘ no aplica por diseño (ver categoría §3.1)

---

## ADRs (Michael Nygard)

Cada decisión `D-NNN` del SPEC tiene un ADR detallado en `docs/adr/`. Formato:

```markdown
# ADR-NNNN · Título

- **Status**: Proposed | Accepted | Superseded | Deprecated
- **Date**: YYYY-MM-DD
- **Scope**: all
- **Implements**: D-NNN
- **Supersedes**: ADR-MMMM (si aplica)

## Context

## Decision

## Alternatives Considered

## Consequences

## References
```

Los ADRs son **inmutables**: si una decisión cambia, crear ADR nuevo con `Supersedes: ADR-NNNN` y marcar el viejo como `Superseded`.

---

## Qué NO hacer

- ❌ **No commitear directamente a `main`** — incluso para typos. Branch protection lo bloquea.
- ❌ **No modificar el SPEC desde otros repos** — los cambios siempre llegan vía PR a este repo.
- ❌ **No eliminar entradas del CHANGELOG ni del SPEC §10** — son históricas. Si una decisión se revierte, marcar como `Deprecated`, no eliminar.
- ❌ **No usar `atlax.ai`** — el dominio canónico es `atlax360.ai` (D-009).
- ❌ **No estimar trabajo en human-hours** — la categoría Atlax trabaja en modo centaur (human+AI pair).
- ❌ **No crear `template-ai-app`** hasta tener ≥2 apps reales pidiendo el template (principio defer + D-010).

---

## Owners

Ver `.github/CODEOWNERS`. Cualquier PR al SPEC requiere review de al menos un owner activo.

Para preguntas sobre el proceso: abrir issue con label `governance` o contactar a `jgcalvo@atlax360.com`.
