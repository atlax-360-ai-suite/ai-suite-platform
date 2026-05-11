# ADR-0008 · NO Edge Functions, SÍ Fluid Compute en Vercel

- **Status**: Accepted
- **Date**: 2026-02
- **Scope**: applicable (apps que despliegan a Vercel)
- **Implements**: D-008

## Context

Vercel ofrece tres runtimes para serverless functions:

| Runtime               | Características                                                                       |
| --------------------- | ------------------------------------------------------------------------------------- |
| **Edge Functions**    | V8 isolate, latencia ultra-baja, sin Node APIs, límite 1MB, sin filesystem            |
| **Serverless** (Node) | Node.js completo, latencia normal, cold starts, límite ~50MB                          |
| **Fluid Compute**     | Híbrido moderno (a partir de 2024): pool de workers Node con cold start ~0, Node APIs |

Las apps de la suite tienen necesidades comunes que chocan con Edge Functions:

- Acceso al filesystem (Bun, lectura de archivos en runtime).
- Conexiones DB persistentes (`postgres`, Drizzle ORM).
- AI SDK con streaming + tools complejos.
- Bundle size frecuentemente >1MB (CSS, fonts, libs).
- Node APIs (`Buffer`, `crypto.randomUUID()`, `node:net`).

Edge Functions, históricamente promovidas por Vercel como "the future", han demostrado limitaciones operativas serias para apps con backend real.

## Decision

**Las apps de la suite que despliegan a Vercel usan Fluid Compute como runtime por defecto. Edge Functions están prohibidas.**

Configuración canónica en `vercel.ts`:

```typescript
export default defineConfig({
  functions: {
    "api/**/*.ts": { runtime: "nodejs22.x" }, // Fluid Compute por defecto
  },
});
```

Verificación: `vercel inspect <deployment>` debe mostrar `runtime: nodejs` para todas las funciones.

**Excepción documentada**: apps que sirven contenido puramente estático (sin endpoints API) pueden usar el runtime que Vercel decida — Edge no aplica porque no hay funciones.

## Alternatives Considered

### Alternativa A — Edge Functions

El default histórico que Vercel empujaba 2022-2024.

- **Pro**: latencia ultra-baja, edge global, ideal para middleware ligero.
- **Contra**: sin Node APIs, sin filesystem, bundle 1MB, sin DB persistente real, debugging difícil, observability limitada. Cada función necesita un workaround para algo que en Node es trivial.
- **Descartada**: tras varios incidentes (incluido un intento real de migración de Kairos en Q4 2025 que se revirtió) la conclusión es que las apps de la suite no son "edge-shaped".

### Alternativa B — Serverless (Node) clásico

El runtime tradicional pre-Fluid.

- **Pro**: Node completo, sin limitaciones.
- **Contra**: cold starts de 1-3 segundos en endpoints poco usados, lo que produce UX degradada y costes (cliente espera, Vercel cobra por wall time).
- **Descartada en favor de Fluid Compute**: Fluid mantiene el mismo modelo de programación pero elimina cold starts.

### Alternativa C — Self-hosted en Cloud Run

Llevar las APIs fuera de Vercel.

- **Pro**: control total, costes predecibles a alto volumen.
- **Contra**: pierde la integración Vercel ↔ GitHub (preview URLs, env vars, AI Gateway); más operación.
- **Descartada para apps web Vercel-native** (Kairos, harvest, dashboard); **aceptada para apps server-only** (atlax-langfuse-bridge stack Langfuse v3 está en Cloud Run).

## Consequences

### Positivas

- **Sin restricciones de bundle**: las apps pueden importar libs grandes sin reescribir.
- **Node APIs disponibles**: `fs`, `path`, `crypto`, `node:net` funcionan.
- **DB connections viables**: pool de conexiones a Postgres real (con `max: 1` en serverless por la naturaleza efímera).
- **Cold start ~0**: Fluid mantiene workers calientes.
- **AI SDK streaming**: tools, structured output y streaming funcionan sin contortions.

### Negativas

- **Latencia ligeramente mayor que Edge** para responses muy pequeñas (50ms vs 5ms). Aceptable porque las apps de la suite no son edge-shaped.
- **Coste algo más alto** que Edge para tráfico muy alto. Mitigación: monitor coste y migrar selectivamente si algún endpoint puro-static se beneficia de Edge (excepción documentada).

### Neutras

- El proxy de Next.js 16.2+ (antes `middleware.ts`, ahora `proxy.ts`) corre en Fluid por default — coherente con la regla.

## References

- [SPEC.md §6.2 — Stack canónico (Deploy API: Fluid Compute, NO Edge Functions)](../SPEC.md#62-stack-canónico-recomendado-no-obligatorio)
- [Vercel Fluid Compute docs](https://vercel.com/docs/functions/runtimes/fluid-compute)
- [ADR-0006 — `vercel.ts` sobre `vercel.json`](./0006-vercel-ts-sobre-vercel-json.md) — define dónde se configura el runtime
- CLAUDE.md de Kairos (regla "Next.js 16.2 usa `proxy.ts`, NO `middleware.ts`")
