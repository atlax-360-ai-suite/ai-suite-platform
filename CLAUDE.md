# CLAUDE.md — ai-suite-platform

Repo de especificación cross-project. Extiende el global `~/.claude/CLAUDE.md`.

## Qué es este repo

Especificación canónica del patrón "Atlax 360 AI Suite — Shared Platform". No contiene código ejecutable.

## Reglas específicas

- **Solo docs**: este repo no tiene código, tests ni dependencias npm. No ejecutar `bun install`.
- **Rama por cambio de versión**: cualquier edit a `docs/SPEC.md` va en rama `docs/spec-vX.Y` y PR con tag `[shared-platform]`.
- **No modificar desde repos de apps**: los cambios al SPEC vienen siempre desde este repo.
- **Dominios canónicos**: `atlax360.ai` y `atlax360.com`. Corregir cualquier mención a `atlax.ai`.
- **Versionado del SPEC**: minor para nuevas decisiones/secciones, patch para correcciones, major para reestructura.

## Identidad git

Este repo pertenece a la org `atlax-360-ai-suite`. La identidad git de Atlax aplica:
`Joserra` / `jgcalvo@atlax360.com`
