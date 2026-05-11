---
name: Reporte de validación de app
about: Reportar el resultado de validar una app de la suite contra el SPEC
title: "[validation] <app-name> contra SPEC vX.Y"
labels: ["governance", "validation"]
assignees: []
---

## Metadata

- **App validada**:
- **Versión del SPEC**: vX.Y
- **Categoría de app**: `web-app` | `edge-tooling` | `server-only`
- **Fecha de validación**: YYYY-MM-DD
- **Validador**: @username

## Resumen ejecutivo

<!-- 3-5 líneas. Cuántos invariantes ✅, cuántos 🟡, cuántos ❌, cuántos ⊘. KP totales, AG totales. -->

## Invariantes evaluados

<!-- Tabla aplicable según §3.1 — marcar cada invariante con ✅/🟡/❌/⊘ -->

| Invariante                      | Estado | Notas |
| ------------------------------- | ------ | ----- |
| Auth-first middleware           |        |       |
| CORS allowlist                  |        |       |
| CSP obligatorio                 |        |       |
| Health endpoint GET /api/health |        |       |
| Correlation ID X-Request-ID     |        |       |
| Logs JSON estructurados         |        |       |
| AbortSignal.timeout en fetch    |        |       |
| Retry exponential backoff       |        |       |
| Workload Identity Federation    |        |       |
| OAuth Client (Google)           |        |       |
| Subdomain `<app>.atlax360.ai`   |        |       |

## Propuestas al SPEC (KP-NN)

<!--
Hallazgos que sugieren un cambio al SPEC canónico.
Si se acepta uno, debe convertirse en PR de `docs/SPEC.md`.
-->

| ID    | Propuesta | Justificación | Severidad    |
| ----- | --------- | ------------- | ------------ |
| KP-01 |           |               | low/med/high |
| KP-02 |           |               |              |

## Deuda de la app (AG-NN)

<!--
Hallazgos que son deuda técnica del proyecto, NO cambios al SPEC.
El proyecto los resuelve en su propio backlog.
-->

| ID    | Gap | Prioridad | Owner |
| ----- | --- | --------- | ----- |
| AG-01 |     |           |       |
| AG-02 |     |           |       |

## Anti-patrones detectados

<!-- Comparar contra §13 del SPEC. ¿La app incurre en alguno? -->

## Snapshot para §11

<!-- Línea exacta que debería aparecer en la tabla §11 del SPEC tras esta validación -->

```
| <app> | <repo> | <deploy> | <auth> | <observability> | <oauth-client> | <subdomain> |
```

## Siguientes pasos

- [ ] Abrir PR `docs/spec-vX.Y` aplicando KP aceptados
- [ ] Actualizar `§11` con nuevo snapshot
- [ ] Abrir tickets de deuda (AG-NN) en el repo de la app
- [ ] Actualizar `HANDOFF-current.md` con resumen de la validación
