# client — `@devdigest/web`

Next.js 15 (App Router) · React 19 · TanStack Query 5 · next-intl 3 · Tailwind 4 ·
recharts 2 · mermaid 11 · Vitest 2 + Testing Library 16 (jsdom). Package manager: **pnpm**.

## Commands

```sh
pnpm dev          # :3000, expects the API on NEXT_PUBLIC_API_BASE (default :3001)
pnpm typecheck
pnpm test         # vitest + jsdom, fetch mocked, no API needed
```

## Where things are

- `src/app/**/page.tsx`: thin route files. Feature UI lives in colocated
  `_components/<Name>/` folders.
- `src/lib/api.ts`: `apiFetch` and `ApiError`, the only place that calls `fetch`.
- `src/lib/hooks/*`: every TanStack Query hook, grouped by domain.
- `src/components/`: cross-page pieces (app shell, diff viewer, page shell).
- `src/vendor/ui/` (`@devdigest/ui`): the design system. Import from the barrel only.
- `messages/en/*.json`: UI strings, one file per feature namespace.

## Conventions

- Component folder layout: `<Name>.tsx`, `index.ts` (re-export, named + default),
  `styles.ts`, `constants.ts`, `helpers.ts`, `<Name>.test.tsx`. Nested children go in
  `<Name>/_components/`.
- Styling is **inline style objects** in `styles.ts` (`export const s = { … }`,
  typed `CSSProperties`), colored only through CSS variables from
  `@devdigest/ui/styles.css`. Don't add Tailwind classes or hex colors in components.
- All user-visible text goes through `next-intl` (`useTranslations`) with keys in
  `messages/en/<namespace>.json`. No hardcoded strings.
- Data access only through a hook in `src/lib/hooks/*`; components never call
  `apiFetch` directly. Invalidate the related query keys after a mutation.
- Add `"use client"` only to components that need state, effects or browser APIs.

## Gotchas

- `src/vendor/shared/` is a **copy** of the server's contracts and lags behind it
  (no `openrouter` provider, no `AgentManifest`). Sync both when you change a contract.
- `messages/en/` already has namespaces for screens that don't exist yet (blast,
  eval, memory…). That's expected; don't delete them.
- Run progress comes over SSE (`/runs/:id/events`); index state polls every 1.5s.
  Don't add polling where an SSE stream exists.

## Do not touch

- `src/vendor/ui/` internals for a one-off need. Extend a primitive or compose
  in `src/components/` instead.

## Read when

- **Read `README.md`** (this folder) when you add a route or a screen that calls
  a new API endpoint; update its route map.
- **Read `src/vendor/ui/README.md`** before building UI, to find an existing
  primitive first.
- **Read `../server/README.md#api-map-starter`** when you need an endpoint's
  shape.
- **Read `specs/`** before building a feature here; write a spec first for a new
  lesson feature (template: `../docs/spec-template.md`).
- **Read `docs/`** (this folder) when you need design notes beyond the README.
- **Read `INSIGHTS.md`** before non-trivial changes here; append new findings to it.
