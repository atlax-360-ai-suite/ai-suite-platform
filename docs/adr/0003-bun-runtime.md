# ADR-0003 · Bun como runtime obligatorio en todos los proyectos Atlax

- **Status**: Accepted (cross-project)
- **Date**: 2026-04
- **Scope**: all
- **Implements**: D-003

## Context

A inicios de 2026 los proyectos Atlax usaban una mezcla de `npm`, `yarn` y `bun`. Eso producía:

- Lockfiles incompatibles entre apps (drift de resoluciones).
- DX fragmentada — devs que rotaban entre proyectos perdían tiempo recordando convenciones.
- CI con setups duplicados (`npm ci` aquí, `bun install --frozen-lockfile` allá).
- Tooling de testing inconsistente (`jest` vs `vitest` vs `bun test`).

Bun 1.3.x se había estabilizado lo suficiente en febrero de 2026:

- Compatibilidad alta con npm packages (incluyendo node-gyp builds).
- Bun test como runner nativo, rápido y sin dependencias.
- Workspaces nativos.
- Significativamente más rápido que npm en install y test.

## Decision

**Bun es el runtime obligatorio para todas las apps de la suite Atlax 360 AI Suite, y por extensión para todos los proyectos del grupo Atlax 360.**

Esto aplica a:

- Instalación de dependencias: `bun install`, nunca `npm install` ni `yarn install`.
- Ejecución de scripts: `bun run <script>` o `bun <file>`, nunca `npm run`.
- Ejecución one-off: `bunx <pkg>`, nunca `npx`.
- Testing: `bun test`, nunca `jest`, `vitest` ni `mocha`.
- Workspaces: `bun` workspaces (`workspace:*`, `catalog:`), no `pnpm` workspaces.

Versión mínima: Bun 1.3.x. Pinear versión en CI vía `oven-sh/setup-bun@v2` con `bun-version: "1.3.x"`.

Lockfile: `bun.lock` (text format) comiteado al repo, con `--frozen-lockfile` en CI.

**Excepciones documentadas**:

- Apps con stack PHP/Python (legacy o por requerimiento de negocio) son legítimas y no aplican esta regla. Pero TypeScript / JavaScript siempre Bun.
- Herramientas que invocan `npx <herramienta-tradicional>` (ej. `npx create-next-app`) → usar `bunx` equivalente. Si la herramienta no funciona con Bun, documentar la excepción puntual.

## Alternatives Considered

### Alternativa A — Mantener `npm` como estándar

Estabilidad probada, ecosistema universal.

- **Pro**: cero migración, máxima compatibilidad.
- **Contra**: lento (instala 3-10× más despacio que Bun), requiere agregar `jest`/`vitest` para testing, requiere tooling de monorepo extra (`turborepo`, `nx`).
- **Descartada**: la velocidad y la simplicidad de Bun superan claramente al ecosistema npm puro.

### Alternativa B — `pnpm`

Buena alternativa a `npm` con symlinking eficiente.

- **Pro**: rápido, workspaces decentes, ecosistema npm intacto.
- **Contra**: sigue necesitando runner aparte para tests; sigue siendo Node bajo el capó; no aporta hot reload moderno.
- **Descartada**: si vamos a romper compatibilidad con `npm`, Bun ofrece beneficios mayores (runtime + runner + bundler integrados).

### Alternativa C — Polyglot (bun para apps + npm para libs)

Permitir Bun en apps pero forzar `npm` en libraries para máxima compatibilidad downstream.

- **Pro**: libraries publicables sin lockfile-incompatibility.
- **Contra**: Bun ya produce `package.json` estándar compatible con `npm`. La incompatibilidad es solo de lockfile, no de output. No hay razón técnica para mantener dos runtimes.
- **Descartada**: añade complejidad sin beneficio.

## Consequences

### Positivas

- **DX coherente**: dev que rota entre apps usa los mismos comandos.
- **CI 3-10× más rápido**: install + test pasan de minutos a segundos.
- **Menos dependencias dev**: no se necesita `jest`, `vitest`, `tsx`, `ts-node` — Bun los reemplaza nativamente.
- **Workspaces simples**: `workspace:*` y `catalog:` cubren los casos cross-package sin tooling adicional.
- **Output cero-deps**: scripts y CLIs pueden bundlearse con `bun build` sin webpack.

### Negativas

- **Algunas libs node-gyp aún rompen** en Bun (raro, pero documentar excepciones).
- **Curva de aprendizaje** para devs viniendo de `npm`/`yarn` (mitigada por docs).
- **`bun` no está en todos los CI runners por defecto** — solucionado con `oven-sh/setup-bun@v2`.

### Neutras

- Lockfile `bun.lock` en text format es diffeable en PRs (ventaja vs `package-lock.json` binario).

## References

- [SPEC.md §6.2 — Stack canónico (Runtime: Bun)](../SPEC.md#62-stack-canónico-recomendado-no-obligatorio)
- [SPEC.md §9.5 — Code quality (Bun para todo)](../SPEC.md#95-code-quality)
- [SPEC.md §13 — Anti-patrones ("Diferentes runtimes entre apps")](../SPEC.md#13-anti-patrones-a-evitar)
- CLAUDE.md global del usuario (regla "ALWAYS use bun")
