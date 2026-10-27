# reviewer-core — `@devdigest/reviewer-core`

Pure review engine: diff → prompt → LLM → structured output → grounded findings.
TypeScript 5.7 (ESM) · Zod 3 · `openai` 4 (OpenRouter client) · Vitest 2.
Package manager: **npm**.

## Commands

```sh
npm install         # needed on its own: zod resolves from this folder's node_modules
npm run typecheck   # this is also the "build"; the package never emits JS
npm test
```

After a change here, also run `pnpm typecheck` in `../server` (its only consumer).

## Where things are

- `src/index.ts`: the public API. Anything the server uses must be exported here.
- `src/review/run.ts`: `reviewPullRequest` orchestration (single-pass / map-reduce).
- `src/prompt.ts`: `assemblePrompt`, `wrapUntrusted`, `INJECTION_GUARD`.
- `src/llm/structured.ts`: Zod → JSON Schema, parse-with-repair.
- `src/grounding.ts`: the citation gate.
- `src/llm/openrouter.ts`: `OpenRouterProvider` (also built by the server's container).

## Rules

- **Stay pure.** No DB, filesystem, GitHub, env vars or `process` access. The only
  side effect is a call through the injected `LLMProvider`. New I/O needs belong
  in `server/`, passed in as inputs.
- **Grounding is mandatory.** Every finding must cite a line present in the diff or
  it is dropped, and the score is recomputed from surviving findings. Never trust
  or pass through the model's own score.
- **Untrusted text is fenced.** Anything from the PR (diff, description, files,
  comments) goes through `wrapUntrusted`. The injection defense is the
  `INJECTION_GUARD` text, by design; don't add keyword filters.
- Optional prompt slots (`skills`, `memory`, `specs`, `callers`, `repoMap`) must be
  omitted from the prompt when empty, so a run without them gives the same
  prompt as before the slot existed.

## Gotchas

- Contracts come from `../server/src/vendor/shared` via a tsconfig path alias, not
  the client copy.
- `reduce()`, `toReview()` and some slots exist for later lessons (CI runner, L06+)
  and aren't wired by the server yet.
- Comments mention an "agent-runner"; it isn't in this repo.

## Read when

- **Read `README.md`** (this folder) when you change the pipeline or public API;
  keep its diagram current.
- **Read `../docs/agent-prompts/README.md`** when you change how prompts are
  assembled or the severity/verdict conventions.
- **Read `specs/`** before building a feature here; write a spec first for a new
  lesson feature (template: `../docs/spec-template.md`).
- **Read `docs/`** (this folder) when you need design notes beyond the README.
- **Read `INSIGHTS.md`** before non-trivial changes here; append new findings to it.
