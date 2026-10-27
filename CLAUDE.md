# DevDigest

Local-first AI pull-request reviewer: import a PR → agent review → grounded,
structured findings. Course starter template: lessons L01–L08 add features back,
so empty tables, unused contracts and i18n keys for missing screens are expected.

## Map

| Path | What | Own rules |
|---|---|---|
| `server/` | Fastify 5 API + Drizzle/Postgres (pgvector), :3001 | `server/CLAUDE.md` |
| `client/` | Next.js 15 / React 19 studio, :3000 | `client/CLAUDE.md` |
| `reviewer-core/` | Pure review engine (prompt → LLM → grounding) | `reviewer-core/CLAUDE.md` |
| `e2e/` | Deterministic browser flows (agent-browser) | `e2e/CLAUDE.md` |
| `docs/` | Cross-package docs: onboarding, agent prompts, spec template | — |
| `scripts/` | `dev.sh` (full local stack), `e2e.sh` (hermetic e2e) | — |

Every package has the same layout for knowledge files: `CLAUDE.md` (rules),
`README.md` (overview), `docs/` (long-form notes), `specs/` (feature specs),
`INSIGHTS.md` (lessons learned). Module-specific conventions live only there, not here.

## Stack and tooling

- Node ≥ 22, TypeScript 5.7 (ESM), Zod 3, Vitest 2, Docker for Postgres.
- **No monorepo workspace.** Each package has its own lockfile and install:
  `server/` and `client/` use **pnpm** (≥ 10); `reviewer-core/` and `e2e/` use **npm**.
  Never run installs from the repo root.
- Packages share code through tsconfig `paths` aliases, not published packages.

## Commands

```sh
./scripts/dev.sh                    # Postgres + migrate + seed + API + web
cd server && pnpm typecheck && pnpm test
cd client && pnpm typecheck && pnpm test
cd reviewer-core && npm run typecheck && npm test
./scripts/e2e.sh                    # isolated stack on :5433/:3101/:3100
```

Run the typecheck and tests of every package you touched before calling work done.
A change to `reviewer-core/` or `server/src/vendor/shared/` also needs `server` typecheck.

## Cross-cutting rules

- `@devdigest/shared` (Zod contracts) exists twice: `server/src/vendor/shared/` and
  `client/src/vendor/shared/`. They have already drifted. A contract change must be
  applied to both copies by hand; reviewer-core reads the server copy.
- READMEs and `docs/` are the source of truth for architecture. Link to them;
  don't copy their content into CLAUDE.md files. If you change a route, pipeline
  or flow, update the matching README diagram in the same change.
- Agent system prompts exist in two places: `docs/agent-prompts/*.md` and
  `server/src/db/seed-prompts.ts`. Edit both.
- Secrets never go into the repo, the DB or logs. They live in
  `~/.devdigest/secrets.json` (fallback: `server/.env`).

## Do not touch

- Applied migrations `server/src/db/migrations/0000–0009*.sql` and `meta/`.
  Change `server/src/db/schema/*` and run `pnpm db:generate` instead.
- Lockfiles by hand (`pnpm-lock.yaml`, `package-lock.json`).
- `.claude/skills/` (third-party skills pinned in `skills-lock.json`).
- `clones/` and `~/.devdigest/` (runtime data).
- Never run `docker compose down -v`: it deletes the dev DB volume with every
  imported repo and review.

## Read when

- **Read `docs/ONBOARDING.md`** when you need the big picture: how the packages
  talk to each other and the review request flow end to end.
- **Read `README.md#architecture`** when a task crosses package boundaries.
- **Read `server/README.md`** when you add or change an API route, env var or
  the review prompt context.
- **Read `reviewer-core/README.md`** when you touch prompt assembly, grounding or
  structured output.
- **Read `server/src/modules/repo-intel/README.md`** when you need repo map,
  file rank, callers or indexing behaviour.
- **Read `docs/agent-prompts/README.md`** when you edit a reviewer prompt or the
  severity/verdict conventions.
- **Read `TESTING.md`** when you add tests or change CI workflows.
- **Read `<module>/specs/`** before implementing a feature: if a spec exists,
  follow it; for a new lesson feature, write one first from `docs/spec-template.md`.
- **Read `<module>/INSIGHTS.md`** before non-trivial work in that module, and
  append to it when you learn something the code doesn't make obvious.
