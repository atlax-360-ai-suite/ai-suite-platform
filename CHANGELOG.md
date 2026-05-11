# Changelog

Todos los cambios significativos al SPEC canónico de Atlax 360 AI Suite — Shared Platform se registran aquí.

El formato sigue [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) y este repo se adhiere a [Semantic Versioning](https://semver.org/lang/es/).

- **major** — reestructura del SPEC o cambio breaking del patrón
- **minor** — nueva decisión D-NNN, nuevo invariante, nueva sección
- **patch** — correcciones, clarificaciones, snapshots de adopción

## [Unreleased]

## [v0.5] — 2026-05-11

### Added

- D-011 nueva: email transaccional interno vía Gmail API + Vercel OIDC + WIF + DWD.
- §9.6 nueva: patrón canónico para email transaccional en apps internas. Sin SendGrid / Resend para destinatarios `@atlax360.com`.
- ADR-0011: justificación técnica + flujo de autenticación de 3 leg (WIF → SA impersonation → DWD JWT bearer) + criterios de migración a proveedor externo.

### Changed

- Header del SPEC bumpeado a v0.5; §10 catálogo con D-011 añadida.

### Notes

- Minor version: nueva decisión D-011 y nuevo invariante §9.6.
- Reference implementation: `kairos/packages/email/` — primera adopción.
- Cuando una 2ª app de la suite adopte el patrón, extraer a `@atlax/email` (repo dedicado, fuera de `ai-suite-platform` que es solo specs).

## [v0.4.1] — 2026-05-11

### Added

- `docs/adr/` con 10 ADRs Michael Nygard formalizando las decisiones D-001..D-010:
  - ADR-0001 Consent Screen y OAuth Clients
  - ADR-0002 Subdominios `<app>.atlax360.ai`
  - ADR-0003 Bun runtime obligatorio
  - ADR-0004 Conventional Commits + Squash merge
  - ADR-0005 Workload Identity Federation GCP↔GitHub
  - ADR-0006 `vercel.ts` sobre `vercel.json`
  - ADR-0007 Vercel AI Gateway por defecto
  - ADR-0008 NO Edge Functions, SÍ Fluid Compute
  - ADR-0009 `atlax360.ai` dominio canónico
  - ADR-0010 Repo dedicado como home del SPEC
- `docs/adr/README.md` con índice + formato canónico Michael Nygard + reglas de inmutabilidad.

### Changed

- `docs/SPEC.md` §10 ahora incluye columna "ADR" con link relativo a cada ADR-NNNN.
- Header del SPEC bumpeado a v0.4.1; footer actualizado.

### Notes

- Patch version (no minor) porque NO se introducen decisiones nuevas — solo formaliza las existentes con el rigor Michael Nygard.
- Los ADRs son inmutables: cambios futuros crean ADR nuevo con `Supersedes:`, no modifican el viejo.

## [v0.4.0] — 2026-05-10

### Added

- Repo dedicado `atlax-360-ai-suite/ai-suite-platform` como home canónico del SPEC (D-010 Accepted).
- Sección 14 actualizada: PRs van a este repo con tag `[shared-platform]`.

### Changed

- Doc movido desde Kairos (`~/work/kairos/docs/atlax-ai-shared-platform.md`) que actuaba como host transitorio hasta v0.3.
- Header del SPEC: `Draft v0.2` → `Accepted v0.4`.
- §10 catálogo de decisiones: D-010 añadida.
- §12 Fase 0 marcada como completada incluyendo creación del repo dedicado.

### Migration notes

- Kairos: 3 ficheros eliminados (`atlax-ai-shared-platform.md`, `HANDOFF-shared-platform-v0.3.md`, `PROMPT-shared-platform-next-session.md`). `docs/00-INDEX.md` actualizado con puntero externo.
- Bridge: `CLAUDE.md` y `docs/audits/shared-platform-validation-2026-05-09.md` apuntan al repo nuevo.

## [v0.3] — 2026-05-09

### Changed

- Corrección del dominio canónico: `atlax360.ai` (no `atlax.ai`) — D-009 Accepted.
- D-002 promovida a Accepted (subdominios `<app>.atlax360.ai`).

### Notas

- Versión publicada en Kairos como host transitorio. Filename original: `docs/atlax-ai-shared-platform.md`.

## [v0.2] — 2026-05-09

### Added

- Consolidación con reporte de validación de `atlax-langfuse-bridge`.
- §3.1 Categorías de aplicación (`web-app`, `edge-tooling`, `server-only`).
- Leyenda canónica de §11 (✅ / 🟡 / ❌ / ⬜ / ⊘).
- KP-01..KP-05 aplicados al SPEC.

### Notas

- Versión publicada en Kairos como host transitorio.

## [v0.1] — 2026-05-09

### Added

- Draft inicial del SPEC generado desde Kairos.
- 5 capas (Plataforma, OAuth Clients, Apps, CI/CD, Portal futuro).
- 8 decisiones iniciales D-001..D-008.
- 9 invariantes transversales no negociables.

### Notas

- Versión publicada en Kairos como host transitorio. Filename original: `docs/atlax-ai-shared-platform.md`.

---

[Unreleased]: https://github.com/atlax-360-ai-suite/ai-suite-platform/compare/v0.5...HEAD
[v0.5]: https://github.com/atlax-360-ai-suite/ai-suite-platform/compare/v0.4.1...v0.5
[v0.4.1]: https://github.com/atlax-360-ai-suite/ai-suite-platform/compare/v0.4.0...v0.4.1
[v0.4.0]: https://github.com/atlax-360-ai-suite/ai-suite-platform/releases/tag/v0.4.0
[v0.3]: https://github.com/atlax-360-ai-suite/ai-suite-platform/compare/v0.2...v0.3
[v0.2]: https://github.com/atlax-360-ai-suite/ai-suite-platform/compare/v0.1...v0.2
[v0.1]: https://github.com/atlax-360-ai-suite/ai-suite-platform/releases/tag/v0.1
