<!--
Recuerda: el título de la PR debe incluir el tag [shared-platform].
Ejemplo: docs(spec): añadir D-011 sobre @atlax/auth [shared-platform]
-->

## Resumen

<!-- 1-3 frases describiendo qué cambia y por qué -->

## Tipo de cambio

- [ ] Cambio al SPEC (`docs/SPEC.md`) — bumpea versión
- [ ] Nuevo ADR Michael Nygard (`docs/adr/`)
- [ ] Actualización de HANDOFF / PROMPT / README
- [ ] Gobernanza del repo (CONTRIBUTING, SECURITY, CODEOWNERS, templates)
- [ ] CI / tooling (`.github/workflows/`, lint configs)
- [ ] Fix tipográfico o de formato

## Decisión asociada (si aplica)

- [ ] Implementa `D-NNN` ya catalogada en SPEC §10
- [ ] Propone nueva decisión `D-NNN` (incluida en este PR)
- [ ] No aplica — gobernanza/tooling sin decisión arquitectónica

## Versionado (si toca el SPEC)

- [ ] **major** — reestructura o breaking change
- [ ] **minor** — nueva decisión / invariante / sección
- [ ] **patch** — corrección / clarificación / snapshot
- [ ] No aplica

Versión propuesta: `vX.Y.Z`

## Apps afectadas

<!-- Marcar las apps de la suite cuya operación se ve impactada por este cambio -->

- [ ] Kairos
- [ ] atlax-langfuse-bridge
- [ ] atlax-claude-dashboard
- [ ] atlax-observatorios
- [ ] harvest
- [ ] Futuras (impacto en el patrón general)

## Checklist

- [ ] Título incluye tag `[shared-platform]`
- [ ] He leído `CONTRIBUTING.md` y sigo el proceso de PR
- [ ] He bumpeado la versión en `docs/SPEC.md` (header + changelog embebido + footer) si el cambio toca el SPEC
- [ ] He añadido entrada en `CHANGELOG.md` con `Added` / `Changed` / `Removed` / `Migration notes`
- [ ] He notificado a owners de las apps afectadas (mencionar en cuerpo del PR)
- [ ] Si introduzco/modifico ADRs, sigo el formato Michael Nygard
- [ ] Si elimino contenido, justifico que es deuda obsoleta (no decisión activa de la suite)
- [ ] He verificado que no introduzco referencias a `atlax.ai` (dominio prohibido — D-009)

## Notas para reviewers

<!-- Contexto adicional, áreas que necesitan atención especial, links a discusiones previas -->
