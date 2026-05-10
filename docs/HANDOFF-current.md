# Handoff — Atlax 360 AI Suite Shared Platform v0.4

**Fecha:** 2026-05-10
**Status:** v0.4 — repo dedicado `atlax-360-ai-suite/ai-suite-platform` activo
**Owner:** jgcalvo@atlax360.com
**Documento canónico:** `docs/SPEC.md` (este repo)
**Scope:** cross-project — kairos, atlax-langfuse-bridge, atlax-claude-dashboard, atlax-observatorios, harvest, futuras apps

---

## Qué ocurrió en esta sesión (2026-05-10)

1. Bridge (`atlax-langfuse-bridge`) completó F1-F5 PRO — PRO activo en `langfuse.atlax360.ai` (924 tests, 0 fail)
2. Se tomó la decisión de promover el doc canónico a repo dedicado (trigger: Kairos ✅ + bridge ✅ ≥ 2 adopciones)
3. Creada org GitHub `atlax-360-ai-suite`
4. Creado repo `atlax-360-ai-suite/ai-suite-platform`
5. Doc v0.3 (host transitorio en Kairos) bumpeado a **v0.4** y movido a `docs/SPEC.md`
6. Kairos actualizado: 3 ficheros eliminados, `docs/00-INDEX.md` actualizado con puntero externo
7. Bridge actualizado: `CLAUDE.md` y `docs/audits/shared-platform-validation-2026-05-09.md` apuntan al repo nuevo

---

## Estado de adopción por proyecto (snapshot 2026-05-10)

| App                    | Categoría    | Subdominio canónico         | Estado adopción                                         | Validación                     |
| ---------------------- | ------------ | --------------------------- | ------------------------------------------------------- | ------------------------------ |
| Kairos                 | web-app      | `kairos.atlax360.ai`        | ✅ activo en PRO; OAuth Client pendiente                | Host histórico del doc         |
| atlax-langfuse-bridge  | edge-tooling | `langfuse.atlax360.ai`      | ✅ activo en PRO; 13/16 invariantes ✅, 0 anti-patrones | ✅ completada 2026-05-09       |
| atlax-claude-dashboard | web-app      | `dashboard.atlax360.ai`     | 🟡 Cloud Run + Keycloak Axesor (NO Google OAuth)        | ⬜ pendiente                   |
| atlax-observatorios    | server-only  | `observatorios.atlax360.ai` | 🟡 interno, deploy TBD                                  | ⬜ pendiente                   |
| harvest                | web-app      | `harvest.atlax360.ai`       | 🟡 Vercel, sprint en curso                              | ⬜ pendiente (sprint en curso) |

---

## Decisiones catalogadas (D-001..D-010)

| ID    | Decisión                                                       | Status                   |
| ----- | -------------------------------------------------------------- | ------------------------ |
| D-001 | Una sola Consent Screen "Atlax 360 AI Suite", N OAuth Clients  | Proposed                 |
| D-002 | Subdominios `<app>.atlax360.ai` canónicos                      | Accepted                 |
| D-003 | Bun como runtime obligatorio                                   | Accepted (cross-project) |
| D-004 | Conventional Commits + Squash merge                            | Accepted (cross-project) |
| D-005 | Workload Identity Federation GCP↔GitHub                        | Proposed                 |
| D-006 | `vercel.ts` sobre `vercel.json`                                | Proposed                 |
| D-007 | Vercel AI Gateway por defecto                                  | Proposed                 |
| D-008 | NO Edge Functions, SÍ Fluid Compute                            | Accepted                 |
| D-009 | `atlax360.ai` canónico; `atlax.ai` NO pertenece al grupo       | Accepted (v0.3)          |
| D-010 | Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home | Accepted (v0.4)          |

---

## Pendiente en bridge (BG-01, BG-04, BG-05)

| ID    | Gap                                                         | Prioridad | Categoría aplicabilidad |
| ----- | ----------------------------------------------------------- | --------- | ----------------------- |
| BG-01 | Sin `PRO_ENV_VARS.md` ni inventario de variables de entorno | Media     | Aplica                  |
| BG-04 | Sin suite de tests automatizados cross-platform             | Media     | Aplica                  |
| BG-05 | Sin `scripts/ops/preflight.sh` de verificación pre-deploy   | Baja      | Aplica                  |

BG-02 y BG-03 fueron descartados (⊘ N/A para edge-tooling).

---

## Trabajo pendiente — Fase 1

- [ ] Validación harvest contra v0.4 (cuando termine sprint en curso)
- [ ] Validación atlax-observatorios contra v0.4
- [ ] Decisión atlax-claude-dashboard: Keycloak vs Google OAuth
- [ ] Bridge: cerrar BG-01/04/05 según prioridad
- [ ] Crear OAuth Client "Kairos" en GCP

---

## Cómo continuar trabajo en este repo en otra sesión

1. Lee este HANDOFF completo
2. Lee `docs/SPEC.md` v0.4 — el documento canónico
3. Lee `docs/PROMPT-next-session.md` para el prompt de arranque

### Tareas válidas en una sesión dedicada

**A. Aplicar feedback de nuevo reporte de validación (harvest, observatorios, dashboard)**

1. Identificar propuestas tipo `KP-NN` → cambios al SPEC
2. Identificar `AG-NN` (App Gap) → deuda del proyecto, NO del SPEC
3. Para cada KP: aplicar al SPEC, bumpear versión, actualizar §10 y §11
4. Actualizar este HANDOFF con nuevo snapshot

**B. Mantenimiento regular del SPEC**

- Actualizar §11 con nuevos snapshots de adopción
- Promover decisiones Proposed → Accepted cuando se cumpla condición
- Catalogar nuevos anti-patrones detectados en validaciones
- Versionado: minor para nuevas decisiones, patch para correcciones, major para reestructura

**C. Crear `template-ai-app`** (cuando ≥2 apps pidan el template)

- Trigger claro: harvest o dashboard pide arrancar app nueva siguiendo el patrón
- Crear `atlax-360-ai-suite/template-ai-app` con workflows CI/CD, CLAUDE.md heredable, `@atlax/auth`

---

## Reglas innegociables de este repo

- **Versionado semántico estricto** — cada cambio bumpea la versión y actualiza el footer del SPEC
- **Tag `[shared-platform]`** en el título de cualquier PR
- **Notación uniforme** §11: ✅ cumple | 🟡 parcial | ❌ deuda | ⬜ no validado | ⊘ no aplica
- **Dominios canónicos**: `atlax360.ai` y `atlax360.com`. Nunca `atlax.ai`
- **NO modificar el SPEC desde repos de apps** — solo desde este repo vía PR
