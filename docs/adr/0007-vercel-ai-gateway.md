# ADR-0007 · Vercel AI Gateway por defecto en lugar de provider-specific SDKs

- **Status**: Proposed
- **Date**: 2026-05-09
- **Scope**: applicable (apps que usan IA en runtime)
- **Implements**: D-007

## Context

Las apps de la suite consumen LLMs (Anthropic Claude, OpenAI GPT, Google Gemini) para tareas como generación de procesos (Kairos), clasificación (harvest), análisis (atlax-observatorios).

Patrones de integración disponibles:

- **SDK provider-específico** (`@anthropic-ai/sdk`, `openai`, `@google/generative-ai`) — llamadas directas al provider.
- **Vercel AI SDK** (`ai`) + AI Gateway — abstracción de proveedor, telemetría, fallback, caching.
- **Langchain / LiteLLM** — abstracciones más completas pero con su propio ecosistema.

Fuerzas en juego:

- **Vendor lock-in**: usar un SDK específico ata la app a ese provider.
- **Telemetría**: trazabilidad de tokens, latencia, errores por endpoint.
- **Cost control**: rate limiting, presupuestos por app.
- **Fallback**: si Anthropic está caído, ¿la app falla o se degrada a OpenAI?
- **Latencia y coste de abstracción**: una capa intermedia añade hops y dependencia.

## Decision

**Vercel AI Gateway + Vercel AI SDK v6 como capa por defecto para llamadas a LLMs desde apps de la suite.**

Patrón canónico:

```typescript
import { generateObject } from "ai";
import { gateway } from "@ai-sdk/gateway";
import { z } from "zod";

const result = await generateObject({
  model: gateway("anthropic/claude-sonnet-4-6"),
  schema: z.object({ ... }),
  prompt: "...",
});
```

Donde `gateway` rutea al provider configurado en el dashboard Vercel AI Gateway, no directo al SDK del provider.

**Apps con casos especializados** (atlax-langfuse-bridge usa LiteLLM por requerimiento de observabilidad; Kairos podría usar Anthropic SDK directo en flujos con tools complejos) son **excepciones documentadas**, no la regla.

## Alternatives Considered

### Alternativa A — `@anthropic-ai/sdk` directo

Llamar al SDK oficial de Anthropic sin capa intermedia.

- **Pro**: latencia mínima, máxima fidelidad a features de Anthropic (tools, prompt caching, batching).
- **Contra**: lock-in con Anthropic; telemetría custom por app; sin fallback automático; cada app implementa rate limit y retry.
- **Aceptado como excepción**: para flujos que dependen de features Anthropic-only (ej. extended thinking, tool use complejo) o donde la latencia es crítica.

### Alternativa B — Langchain

Abstracción rica con muchas integraciones.

- **Pro**: ecosistema amplio, agents, memory, vector stores integrados.
- **Contra**: superficie de API enorme, breaking changes frecuentes, calidad inconsistente entre integraciones. Para la suite (que necesita 80% generación estructurada simple), es over-tooling.
- **Descartada**: vendor lock-in en abstracción + complejidad operacional desproporcionada.

### Alternativa C — LiteLLM (proxy unified API)

Proxy que expone una API OpenAI-compatible para 100+ providers.

- **Pro**: vendor-neutral, self-hosted, total control.
- **Contra**: añade un servicio que mantener; latencia extra; sin integración nativa con Vercel telemetry.
- **Aceptada como excepción para atlax-langfuse-bridge** que ya tiene LiteLLM por razones de observabilidad PRO.

### Alternativa D — Cliente custom por app

Cada app implementa su propio cliente HTTP a los providers.

- **Pro**: control total.
- **Contra**: reimplementar retry, rate limit, telemetría, schema validation por app. Duplica esfuerzo.
- **Descartada**: anti-DRY.

## Consequences

### Positivas

- **Vendor agnostic** a nivel de código: cambiar de Sonnet a Opus o de Anthropic a Gemini es config, no código.
- **Telemetría integrada**: Vercel AI Gateway expone tokens, latencia, errores en su dashboard.
- **Fallback automático**: configurable en el gateway sin tocar código.
- **Rate limit y budget** centralizado a nivel de gateway, no app.
- **Coherencia cross-app**: si todas las apps usan el mismo patrón, el equipo aprende una sola API.

### Negativas

- **Capa adicional**: 5-50ms extra de latencia por request.
- **Dependencia de Vercel**: si Vercel AI Gateway tiene outage, todas las apps que dependen se ven afectadas. Mitigación: fallback a SDK directo en `try/catch`.
- **Coste extra del Gateway** (a partir de cierto volumen Vercel cobra el routing). Monitorear.
- **Features avanzadas del provider** (prompt caching de Anthropic, batching, extended thinking) pueden no estar expuestas en la capa de abstracción. Mitigación: usar SDK directo como excepción documentada.

### Neutras

- Los modelos se referencian con strings (`"anthropic/claude-sonnet-4-6"`) — riesgo de typo, mitigado con constantes en `@<app>/types`.

## References

- [SPEC.md §6.2 — Stack canónico (AI provider: Anthropic via Vercel AI Gateway)](../SPEC.md#62-stack-canónico-recomendado-no-obligatorio)
- [Vercel AI SDK v6 docs](https://sdk.vercel.ai/docs)
- [Vercel AI Gateway docs](https://vercel.com/docs/ai-gateway)
- CLAUDE.md global del usuario (sección "Model Cost Optimization") — define qué modelo usar por tipo de tarea
