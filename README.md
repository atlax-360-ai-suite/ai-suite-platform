# Atlax 360 AI Suite — Shared Platform

Especificación canónica del patrón de plataforma compartida para la categoría de aplicaciones **Atlax 360 AI Suite** — apps Atlax que integran IA agéntica.

## ¿Qué hay aquí?

| Fichero                                                        | Propósito                                                                  |
| -------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [`docs/SPEC.md`](./docs/SPEC.md)                               | Especificación completa v0.4 — 5 capas, invariantes, decisiones, roadmap   |
| [`docs/HANDOFF-current.md`](./docs/HANDOFF-current.md)         | Estado actual: adopciones por proyecto, pendientes, decisiones catalogadas |
| [`docs/PROMPT-next-session.md`](./docs/PROMPT-next-session.md) | Prompt de arranque para sesiones dedicadas a este repo                     |

## Apps en la suite

| App                    | Categoría    | Subdominio                  | Estado       |
| ---------------------- | ------------ | --------------------------- | ------------ |
| Kairos                 | web-app      | `kairos.atlax360.ai`        | ✅ PRO       |
| atlax-langfuse-bridge  | edge-tooling | `langfuse.atlax360.ai`      | ✅ PRO       |
| atlax-claude-dashboard | web-app      | `dashboard.atlax360.ai`     | 🟡 parcial   |
| atlax-observatorios    | server-only  | `observatorios.atlax360.ai` | ⬜ pendiente |
| harvest                | web-app      | `harvest.atlax360.ai`       | 🟡 en curso  |

## Cómo proponer cambios

1. PR contra este repo modificando `docs/SPEC.md`
2. Tag `[shared-platform]` en el título
3. Notificar a owners de proyectos afectados
4. Consenso de owners antes de merge

## Dominios canónicos

`atlax360.ai` y `atlax360.com`. Nunca `atlax.ai` (no pertenece al grupo — D-009).
