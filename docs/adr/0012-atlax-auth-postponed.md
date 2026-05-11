# ADR-0012 · `@atlax/auth` postponed (insufficient consumer overlap)

- **Status**: Proposed
- **Date**: 2026-05-11
- **Scope**: all
- **Implements**: D-012

## Context

La Capa 4 del SPEC (§7.1) contempla un `packages/@atlax/auth` dentro de `template-ai-app` que
cubra Supabase Auth, Google OAuth y RBAC para cualquier nueva app de la suite. La pregunta
natural, una vez que Kairos está en PRO y Orvian tiene auth maduro, es si extraer ese package
de forma retroactiva a partir del código existente.

El 2026-05-11 se realizó una auditoría en los 4 proyectos con autenticación activa: Kairos,
Orvian, Harvest y atlax-claude-dashboard. El objetivo era medir cuánto código es realmente
compartible y si el coste de gobernanza se justifica.

## Decision

**Postponer la extracción de `@atlax/auth` como package shared-platform. Cada app mantiene su
propia implementación de auth hasta que se cumplan los criterios de reactivación documentados
en este ADR.**

La intersección compartible es pequeña (~150 líneas), los modelos de autenticación son
arquitectónicamente incompatibles entre los proyectos de mayor valor, y la infraestructura
para publicar y consumir paquetes `@atlax/*` no existe.

## Evidence (auditoría 2026-05-11)

### Kairos (`~/work/kairos`)

- Supabase SSR con `@supabase/ssr` (`createServerClient` / `createBrowserClient`). Next 16.2
  con `proxy.ts` para session refresh y redirect a `/login`.
- `getAuthContext()` en `apps/web/src/lib/auth.ts` (L28-62): discriminated union `{ok: true,
ctx}` / `{ok: false, response}`. El `AuthContext` incluye `db` inyectada — acoplamiento
  directo con el schema Drizzle de Kairos y la fase RLS activa.
- RBAC config-driven en código: `@kairos/types/rbac.ts`, matrix `ROLE_PERMISSIONS`, 4 roles
  con wildcards `*:*` / `resource:*`.
- Google OAuth cableado en `(auth)/callback/route.ts` pero OAuth Client en GCP pendiente —
  diferido a v2.16.

### Orvian (`~/work/orvian`)

- Mismas factories `@supabase/ssr` pero el `signIn()` pasa por un **identity-service Hono**
  (Cloudflare Worker propio) que luego inyecta tokens vía `setSession()`. Flujo radicalmente
  distinto al de Kairos (directo a Supabase).
- Tres packages propios: `@orvian/auth` (contratos provider-neutral + adapters Supabase),
  `@orvian/service-auth` (tokens M2M), `@orvian/service-tenant` (resolución tenant/workspace
  por header). Arquitectura interna madura que no encaja en una interfaz simplificada.
- RBAC **en base de datos**: tablas `platform.rbac_roles`, `rbac_policies`, `rbac_resources`,
  `rbac_actions`, `rbac_user_roles`. Evaluador con impersonación. Incompatible con el modelo
  constants-in-code de Kairos.
- SSO real con **Microsoft Azure AD** (`provider: 'azure'`), no Google OAuth.
- Nota crítica: `getAuthConfig()` lee `IDENTITY_SERVICE_URL` con múltiples fallbacks a
  `CORE_API_URL` y `PLATFORM_API_URL` — deuda que `@atlax/auth` no debe heredar.

### Harvest (`~/work/harvest`)

- Stack distinto: Hono API + Vite PWA. Sin Next.js App Router, sin `proxy.ts`, sin `withAuth`.
- Dos mecanismos paralelos: API Keys M2M (validadas contra tabla `api_keys`) + Supabase JWT
  Bearer en la PWA. Server-side con `service_role_key` (sin RLS).
- Google OAuth es **OAuth de integración** (Gmail/Drive scopes para captura de facturas), no
  login. Tokens en Supabase Vault.
- RBAC inexistente: autorización app-level por endpoint.

### atlax-claude-dashboard (`~/work/atlax-claude-dashboard`)

- **NextAuth v5 con provider Keycloak Axesor** — sin Supabase Auth en absoluto.
- `middleware.ts` (no `proxy.ts`). PostgreSQL directo + Drizzle sin cliente Supabase.
- Restricción de dominio `@atlax360.com` en `signIn` callback; RBAC inexistente más allá
  de ese gate.

### Infraestructura `@atlax/*`

- `grep "@atlax/"` en los 4 repos: **0 matches**. El namespace no existe en ningún consumidor.
- `CLAUDE.md` de este repo (línea 11) explicita que `ai-suite-platform` es solo docs — sin
  código, tests ni dependencias npm.
- `gh repo list atlax-360-ai-suite` devuelve 1 repo: `ai-suite-platform`. Sin
  `template-ai-app`, sin registry privado, sin `.npmrc` con registry externo en ningún
  proyecto.

## Alternatives Considered

### Alternativa A — Extraer ahora con Kairos como único consumidor real

Crear `@atlax/auth` con las factories Supabase + proxy helper + `withAuth` de Kairos.

**Rechazada**: YAGNI. Un package con un único consumidor que es el propio origen de la
extracción no reduce duplicación, solo añade indirección. Cualquier cambio en Kairos exige
versionar y publicar el package antes de volver a consumirlo. Overhead neto negativo.

### Alternativa B — Extraer solo factories + proxy helper (~150 líneas)

La única intersección real entre Kairos y Orvian son las llamadas `createServerClient()` /
`createBrowserClient()` y el pattern de `proxy.ts`.

**Rechazada**: ~150 líneas con coste de gobernanza completo (versioning semántico cross-repo,
breaking changes, CI de publish, CHANGELOG propio, consumidores bloqueados en upgrades).
El beneficio DRY no supera ese coste. Además los dos proxies ya divergen: Orvian incluye i18n

- OTLP spans; Kairos es redirect simple. La abstracción útil sería demasiado delgada para ser
  mantenible.

### Alternativa C — Esperar a `template-ai-app` operativo y 3er consumidor confirmado _(aceptada)_

Si se construye `template-ai-app` (SPEC §7.1), `@atlax/auth` existe por construcción del
template, no por extracción retroactiva. El package se diseña para el caso nuevo sin necesidad
de encajar en tres implementaciones divergentes ya existentes. Alineado con el principio rector
§2 #6: "Defer cuando puedas".

## Consequences

### Positivas

- Cero overhead de gobernanza sobre implementaciones que evolucionan rápido (Kairos en sprint
  semanal, Orvian con arquitectura en maduración).
- Cada proyecto rompe su API de auth sin coordinar con un package compartido.
- No se hereda la deuda de Orvian (`IDENTITY_SERVICE_URL` + múltiples fallbacks) en el package
  canónico.
- La decisión es reversible: ningún código existente se elimina ni se bloquea la futura
  extracción.

### Negativas

- Duplicación controlada persiste: factories Supabase y el pattern de `proxy.ts` se mantienen
  en cada repo por separado.
- Onboarding de nueva app requiere copiar el boilerplate de auth manualmente hasta que exista
  `template-ai-app`.

### Neutras

- La Capa 4 del SPEC (§7.1) permanece como diseño target — `@atlax/auth` está especificado,
  solo no construido todavía.
- La referencia a `packages/@atlax/auth/` en el diagrama del template es aspiracional; ningún
  consumidor actual está bloqueado por su ausencia.

## Reactivation criteria

Reabrir cuando se cumpla **cualquiera** de los siguientes:

1. **Aparece una 4ª app Atlax con stack idéntico a Kairos** (Supabase SSR + Next 16 + RBAC
   constants-in-code + single-workspace). En ese momento la intersección sube de 2 a 3
   consumidores y la abstracción se justifica sin forzar.
2. **Orvian elimina su identity-service Hono y migra a Supabase directo**, alineando el flujo
   de `signIn()` con Kairos. Eso convierte a Orvian en consumidor real del package.
3. **Se construye `template-ai-app`** (SPEC §7.1): `@atlax/auth` existe por diseño del
   template. El package se diseña para el caso nuevo, sin retroadaptación.
4. **Infraestructura de publicación operativa**: registry (GitHub Packages, npm privado o
   `git+https`), CI de publish con semver automatizado, y al menos un consumidor distinto del
   autor del package.

Reabrir requiere PR a este repo con un ADR nuevo (0013 o superior) con campo
`Supersedes: ADR-0012`.

## References

- [SPEC.md §2 — Principio rector #6 (Defer cuando puedas)](../SPEC.md#2-principios-rectores)
- [SPEC.md §7.1 — Template repo con `@atlax/auth` aspiracional](../SPEC.md#71-template-repo)
- [ADR-0010 — Repo dedicado es solo specs, sin código npm](./0010-repo-dedicado-ai-suite-platform.md)
- Archivos auditados:
  - `kairos/apps/web/src/lib/auth.ts` (L28-62) — `getAuthContext()` y `AuthContext` con `db`
  - `kairos/packages/@kairos/types/src/rbac.ts` — `ROLE_PERMISSIONS` matrix
  - `kairos/apps/web/src/proxy.ts` — session refresh + redirect
  - `orvian/packages/@orvian/auth/` — contratos provider-neutral + adapters Supabase
  - `orvian/apps/web/src/proxy.ts` — i18n + OTLP spans sobre session refresh
  - `harvest/apps/api/src/middleware/` — Hono middleware auth + organization enforcement
  - `atlax-claude-dashboard/apps/dashboard/src/auth.ts` — NextAuth v5 + Keycloak config
