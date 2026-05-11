# ADR-0006 · `vercel.ts` sobre `vercel.json` cuando exista config

- **Status**: Proposed
- **Date**: 2026-05-09
- **Scope**: applicable (apps que despliegan a Vercel)
- **Implements**: D-006

## Context

Vercel ofrece configuración de proyecto en dos formatos:

- **`vercel.json`** — formato histórico, JSON puro. No permite cómputo, comentarios, ni reuso entre entornos.
- **`vercel.ts`** — formato moderno (a partir de Vercel CLI 35+), TypeScript con tipado. Permite cómputo, condicionales por entorno, importación de constantes desde código, comentarios.

Las apps de la suite tienen necesidades comunes que se benefician de cómputo en la config:

- Headers de seguridad (CSP, HSTS) que dependen de variables de entorno.
- Rewrites condicionales según `VERCEL_ENV` (`preview` vs `production`).
- Reuso de constantes entre el código y la config de deploy (`MAX_BODY_SIZE`, allowlist de orígenes CORS).
- Tipado: detectar typos antes del deploy.

## Decision

**Cuando una app de la suite necesite configuración de Vercel más allá del default, usar `vercel.ts` en lugar de `vercel.json`.**

`vercel.json` se acepta cuando:

- La app no necesita config (greenfield, defaults son suficientes).
- Migración progresiva desde un estado previo.

`vercel.ts` se exige cuando:

- Hay headers de seguridad customizados.
- Hay rewrites o redirects condicionales.
- Se importan constantes desde código.
- La config supera ~20 líneas.

Patrón canónico (template):

```typescript
import { defineConfig } from "vercel";
import { CORS_ALLOWLIST, MAX_BODY_SIZE_BYTES } from "@<app>/types";

export default defineConfig({
  buildCommand: "bun run build",
  framework: "nextjs",
  headers: [
    {
      source: "/(.*)",
      headers: [
        { key: "Strict-Transport-Security", value: "max-age=63072000" },
        { key: "X-Content-Type-Options", value: "nosniff" },
        {
          key: "Content-Security-Policy",
          value: buildCsp({
            env: process.env.VERCEL_ENV,
            origins: CORS_ALLOWLIST,
          }),
        },
      ],
    },
  ],
});
```

## Alternatives Considered

### Alternativa A — Mantener `vercel.json` siempre

Status quo.

- **Pro**: cero migración.
- **Contra**: imposibilita reuso de constantes; CSP estática se vuelve incoherente si cambia el código; sin tipado.
- **Descartada**: el coste de drift entre código y config crece con el tiempo.

### Alternativa B — Generar `vercel.json` desde un script TS al build

`bun run scripts/build-vercel-config.ts > vercel.json` antes del deploy.

- **Pro**: el output es JSON estándar, compatible con cualquier herramienta que lo lea.
- **Contra**: dos artefactos que mantener (el script + el output); el output debe estar en git o generarse en CI; más fricción.
- **Descartada**: `vercel.ts` ya hace esto de forma nativa.

### Alternativa C — Usar Turborepo / Nx para config compartida

Centralizar config en un package y consumir desde cada app.

- **Pro**: reuso máximo cross-app.
- **Contra**: la config Vercel es app-specific por naturaleza; el reuso real es de helpers (`buildCsp()`), no de la config completa. Esos helpers ya viven en `@atlax/*` packages.
- **Descartada**: over-engineering.

## Consequences

### Positivas

- **Tipado**: typos detectados antes del deploy (`buildCommnad` → error de compilación).
- **Reuso de constantes**: la CSP allowlist es la misma en código y deploy config — fuente única.
- **Condicionales por entorno**: `if (env === "preview") { ... }` legible.
- **Comentarios**: posible explicar decisiones inline.
- **Migración suave**: convivir `vercel.ts` con `vercel.json` hasta que se elimine el segundo.

### Negativas

- **Versión Vercel CLI ≥35** requerida. Mitigación: pinear versión en CI.
- **Bundle size del config**: si se importa código pesado en `vercel.ts`, el deploy puede ralentizarse. Mitigación: importar solo tipos y constantes.

### Neutras

- Es opcional: apps sin config solo necesitan defaults.

## References

- [SPEC.md §7.1 — Template repo (vercel.ts canónico)](../SPEC.md#71-template-repo)
- [Vercel Docs — TypeScript config](https://vercel.com/docs/concepts/projects/project-configuration)
- [ADR-0008 — Fluid Compute](./0008-no-edge-functions-fluid-compute.md) — config Vercel define el runtime target
