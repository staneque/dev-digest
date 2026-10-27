# server — `@devdigest/api`

Fastify 5.2 · Drizzle ORM 0.38 / drizzle-kit 0.30 · postgres-js · pgvector ·
`fastify-type-provider-zod` 4 · fastify-sse-v2 · p-queue 8 · `@ast-grep/napi` 0.43 ·
Vitest 2 + testcontainers 10. Package manager: **pnpm**.

## Commands

```sh
pnpm dev                                          # tsx watch, :3001
pnpm typecheck
pnpm exec vitest run --exclude '**/*.it.test.ts'  # unit, no Docker
pnpm exec vitest run .it.test                     # integration, needs Docker
pnpm db:generate && pnpm db:migrate               # after editing src/db/schema/*
pnpm db:seed                                      # idempotent demo data
```

## Where things are

- `src/app.ts` builds the app (plugins → container → modules → error handler).
- `src/modules/<name>/`: `routes.ts` (Fastify plugin), `service.ts`,
  `repository.ts`, `helpers.ts`, `constants.ts`. Registered by hand in
  `src/modules/index.ts`; there is no autoload.
- `src/platform/`: DI container, job runner, SSE bus, config, errors.
- `src/adapters/`: LLM, GitHub, git, ast-grep, tokenizer, secrets; test doubles
  in `src/adapters/mocks.ts`.
- `src/db/schema/*` (one file per domain) re-exported from `src/db/schema.ts`.

## Conventions

- Get every external dependency from `app.container` (`container.llm()`,
  `container.github()`, `container.repoIntel`, …). Never `new` an adapter in a
  module, or tests can't inject mocks.
- Routes declare Zod `params`/`body` schemas through
  `app.withTypeProvider<ZodTypeProvider>()`; don't hand-parse `req.body`.
- Throw `AppError` subclasses from `src/platform/errors.ts`; the shared handler
  turns them into the error envelope.
- Long work goes through `container.jobs` (register a kind, then `enqueue`), not
  un-awaited promises in a route. Review runs are the one exception
  (`ReviewRunExecutor`).
- Local imports use the `.js` extension (ESM).

## Gotchas

- Migrations do **not** run on boot. A `relation … does not exist` error means
  `pnpm db:migrate`.
- A test that imports `test/helpers/pg.ts` **must** be named `*.it.test.ts`,
  or CI runs it in the Docker-less unit job and it fails.
- `src/platform/{grounding,prompt,structured}.ts` are re-export shims. The logic
  is in `reviewer-core/`; change it there.
- `runBus` and the job queue are in-memory; the boot reaper assumes one API
  process per DB. Restarting marks in-flight runs as failed.
- Repo-intel degrades silently: an unindexed repo gives a diff-only prompt.
- After storing a new key, call `container.invalidateSecretCaches()`.
- `NODE_ENV=test` disables the global rate limit; per-route limits still apply.

## Do not touch

- Applied migration files and `src/db/migrations/meta/`.
- `src/vendor/shared/` without mirroring the change into `client/src/vendor/shared/`.

## Read when

- **Read `README.md`** (this folder) when you add a route, env var or module, or
  change what goes into the review prompt.
- **Read `src/modules/repo-intel/README.md`** when you work on indexing or the
  `repoIntel.*` facade.
- **Read `../TESTING.md`** when you add a test suite or touch CI.
- **Read `specs/`** before building a feature here; write a spec first for a new
  lesson feature (template: `../docs/spec-template.md`).
- **Read `docs/`** (this folder) when you need design notes beyond the README.
- **Read `INSIGHTS.md`** before non-trivial changes here; append new findings to it.
