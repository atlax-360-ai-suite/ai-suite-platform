# Atlax 360 AI Suite — Shared Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/SPEC-v0.4-brightgreen.svg)](./docs/SPEC.md)
[![Status](https://img.shields.io/badge/status-Accepted-success.svg)](./docs/SPEC.md)

Especificación canónica del patrón de plataforma compartida para la categoría de aplicaciones **Atlax 360 AI Suite** — apps Atlax que integran IA agéntica.

## ¿Qué hay aquí?

| Fichero                                                        | Propósito                                                                  |
| -------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [`docs/SPEC.md`](./docs/SPEC.md)                               | Especificación completa v0.4 — 5 capas, invariantes, decisiones, roadmap   |
| [`docs/adr/`](./docs/adr/)                                     | ADRs Michael Nygard formalizando las decisiones D-001..D-010               |
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

1. Lee [`CONTRIBUTING.md`](./CONTRIBUTING.md) para el proceso completo
2. Abre PR contra este repo modificando `docs/SPEC.md` (o creando ADR en `docs/adr/`)
3. Título con tag `[shared-platform]`
4. Notifica a owners (ver [`CODEOWNERS`](./.github/CODEOWNERS))
5. Consenso de owners antes de merge

## Gobernanza

| Documento                              | Propósito                                         |
| -------------------------------------- | ------------------------------------------------- |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Proceso de PR, versionado semántico, convenciones |
| [`CHANGELOG.md`](./CHANGELOG.md)       | Historial de versiones del SPEC (v0.1..v0.4)      |
| [`SECURITY.md`](./SECURITY.md)         | Política de reporte de vulnerabilidades           |
| [`LICENSE`](./LICENSE)                 | MIT                                               |

## Dominios canónicos

`atlax360.ai` y `atlax360.com`. Nunca `atlax.ai` (no pertenece al grupo — D-009).
