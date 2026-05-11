# Handoff — Atlax 360 AI Suite Shared Platform v0.6

**Fecha:** 2026-05-11
**Status:** v0.6 — ADR-0012 `@atlax/auth` postponed
**Owner:** jgcalvo@atlax360.com
**Documento canónico:** `docs/SPEC.md` (este repo)
**Scope:** cross-project — kairos, atlax-langfuse-bridge, atlax-claude-dashboard, atlax-observatorios, harvest, futuras apps

---

## Qué ocurrió en la sesión 2026-05-11

1. **Gobernanza base** (PR #1): LICENSE, CHANGELOG, CONTRIBUTING, SECURITY, PR template, issue templates, CODEOWNERS, README badges
2. **ADRs Michael Nygard** (PR #2): 10 ADRs formalizando D-001..D-010 en `docs/adr/`; SPEC bumpeado a v0.4.1 con columna ADR en §10
3. **Bridge descontaminación** (rama directa): JSDoc comment de `naming-convention.test.ts` línea 158 actualizado — referencia al doc Kairos transitorio sustituida por puntero al repo canónico
4. **Branch protection** activada en `main`: force-push y delete bloqueados; PR review requerida
5. **Tags y releases**: `v0.4.0` (retroactivo, commit inicial) y `v0.4.1` (ADRs) creados y publicados como GitHub Releases
6. **CI** (PR #3): `docs-ci.yml` con markdownlint-cli2 + lychee link checker; `.markdownlint.json`; `.github/lychee.toml` — ambos checks pasan en main
7. **ADR-0011** (PR #5): D-011 email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD. SPEC bumpeado a v0.5.
8. **ADR-0012** (PR #6 — esta PR): D-012 `@atlax/auth` postponed tras auditoría de 4 proyectos. SPEC bumpeado a v0.6.

---

## Qué ocurrió en la sesión anterior (2026-05-10)

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

## Estado de infraestructura del repo (snapshot 2026-05-11)

| Componente        | Estado                                                            |
| ----------------- | ----------------------------------------------------------------- |
| SPEC              | v0.6 — `docs/SPEC.md`                                             |
| ADRs              | `docs/adr/` — 12 ADRs (0001..0012), todos con Scope tag           |
| CI                | ✅ markdown lint + link check activos en PRs y main               |
| Branch protection | ✅ main protegida — no force-push, no delete, PR review requerida |
| Tags / Releases   | ✅ v0.4.0, v0.4.1 en GitHub Releases                              |
| CODEOWNERS        | `@joserragonzalez` — único maintainer                             |
| Bridge cleanup    | ✅ JSDoc naming-convention.test.ts actualizado                    |

---

## Decisiones catalogadas (D-001..D-012)

| ID    | Decisión                                                                          | Status                   |
| ----- | --------------------------------------------------------------------------------- | ------------------------ |
| D-001 | Una sola Consent Screen "Atlax 360 AI Suite", N OAuth Clients                     | Proposed                 |
| D-002 | Subdominios `<app>.atlax360.ai` canónicos                                         | Accepted                 |
| D-003 | Bun como runtime obligatorio                                                      | Accepted (cross-project) |
| D-004 | Conventional Commits + Squash merge                                               | Accepted (cross-project) |
| D-005 | Workload Identity Federation GCP↔GitHub                                           | Proposed                 |
| D-006 | `vercel.ts` sobre `vercel.json`                                                   | Proposed                 |
| D-007 | Vercel AI Gateway por defecto                                                     | Proposed                 |
| D-008 | NO Edge Functions, SÍ Fluid Compute                                               | Accepted                 |
| D-009 | `atlax360.ai` canónico; `atlax.ai` NO pertenece al grupo                          | Accepted (v0.3)          |
| D-010 | Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home                    | Accepted (v0.4)          |
| D-011 | Email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD (no SendGrid) | Proposed                 |
| D-012 | `@atlax/auth` postponed — cada app mantiene su auth hasta criterios reactivación  | Proposed                 |

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

- [ ] Validación harvest contra v0.6 (cuando termine sprint en curso)
- [ ] Validación atlax-observatorios contra v0.6
- [ ] Decisión atlax-claude-dashboard: Keycloak vs Google OAuth
- [ ] Bridge: cerrar BG-01/04/05 según prioridad
- [ ] Crear OAuth Client "Kairos" en GCP
- ✅ ~~Decisión `@atlax/auth`~~ — cerrada vía ADR-0012 (postponed); reabrir con ADR-0013+ cuando se cumplan criterios de reactivación

---

## Cómo continuar trabajo en este repo en otra sesión

1. Lee este HANDOFF completo
2. Lee `docs/SPEC.md` v0.6 — el documento canónico
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
- Crear `atlax-360-ai-suite/template-ai-app` con workflows CI/CD, CLAUDE.md heredable, `@atlax/auth` pre-cableado (ver ADR-0012 §Reactivation criteria #3 — es uno de los triggers para reactivar la extracción)

---

## Reglas innegociables de este repo

- **Versionado semántico estricto** — cada cambio bumpea la versión y actualiza el footer del SPEC
- **Tag `[shared-platform]`** en el título de cualquier PR
- **Notación uniforme** §11: ✅ cumple | 🟡 parcial | ❌ deuda | ⬜ no validado | ⊘ no aplica
- **Dominios canónicos**: `atlax360.ai` y `atlax360.com`. Nunca `atlax.ai`
- **NO modificar el SPEC desde repos de apps** — solo desde este repo vía PR
