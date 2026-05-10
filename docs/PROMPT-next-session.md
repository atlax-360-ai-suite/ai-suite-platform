# Prompt — Sesión dedicada Atlax 360 AI Suite Shared Platform

> Este prompt está pensado para sesiones que trabajen el SPEC cross-project.
> El doc canónico vive en `~/work/ai-suite-platform/docs/SPEC.md` (repo `atlax-360-ai-suite/ai-suite-platform`).

---

## Pega esto en una sesión limpia

Estoy arrancando una sesión dedicada al trabajo cross-project del documento "Atlax 360 AI Suite — Patrón Shared Platform".

**Contexto:** este documento define el patrón de plataforma compartida para la categoría de apps "Atlax 360 AI Suite". Vive en el repo dedicado `atlax-360-ai-suite/ai-suite-platform` (clonado en `~/work/ai-suite-platform/`).

**Lee primero, en orden:**

1. `~/work/ai-suite-platform/docs/HANDOFF-current.md` — estado completo, decisiones tomadas, roadmap, validaciones por proyecto
2. `~/work/ai-suite-platform/docs/SPEC.md` — documento canónico v0.4
3. `~/work/ai-suite-platform/README.md` — punto de entrada del repo

**Estado actual (snapshot 2026-05-10):**

- v0.4 publicada — repo dedicado activo, ya no host transitorio en Kairos
- Kairos ✅ y bridge ✅ — 2 adopciones formales que triggerearon el repo dedicado
- Pilotaje bridge PRO con 13 devs arrancando 2026-05-11
- Pendiente: validación harvest, atlax-claude-dashboard, atlax-observatorios

**Posibles tareas:**

### A. Aplicar feedback de reporte de validación de un proyecto

Cuando llegue reporte de validación de harvest / observatorios / dashboard:

1. Identificar `KP-NN` (propuestas de cambio al SPEC) vs `AG-NN` (deuda del proyecto)
2. Para cada KP: aplicar en `docs/SPEC.md`, bumpear versión, actualizar §10 y §11
3. Actualizar `docs/HANDOFF-current.md` con nuevo snapshot
4. PR con tag `[shared-platform]`

### B. Mantenimiento regular del SPEC

- Actualizar §11 con snapshots de adopción
- Promover decisiones Proposed → Accepted
- Catalogar nuevos anti-patrones

### C. Crear `template-ai-app`

Trigger: harvest o dashboard arranca app nueva siguiendo el patrón.
Crear `atlax-360-ai-suite/template-ai-app` con CI/CD workflows, CLAUDE.md heredable, `@atlax/auth`.

---

## Antes de empezar

1. Confirma con el usuario qué tarea (A/B/C u otra)
2. Si vas a modificar el SPEC, crea rama `docs/spec-vX.Y` antes de cualquier edit
3. Si vas a crear `template-ai-app`, confirma que el trigger se cumple (app nueva real, no hipotética)

¿Qué tarea abordamos?
