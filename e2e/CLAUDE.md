# e2e — `@devdigest/e2e`

Deterministic browser flows driven by the Vercel **agent-browser** CLI (Rust + CDP).
No Playwright, no LLM, no API key. TypeScript runner via `tsx`. Package manager: **npm**.

## Commands

```sh
npm i -g agent-browser && agent-browser install   # once
../scripts/e2e.sh        # recommended: isolated seeded stack on :5433/:3101/:3100
npm test                 # against a running stack (E2E_BASE_URL, default :3000)
npm run typecheck
```

## Where things are

- `specs/NN-name.flow.json`: one flow per file, ordered list of agent-browser commands.
- `run.ts`: runs every flow in one shared browser session.
- `lib/assert.ts`: the optional `stdoutIncludes` check.

## Rules

- Each `cmd` is passed verbatim to agent-browser; a non-zero exit fails the flow.
  `wait --text` / `wait --url` **are** the assertions.
- Use deterministic locators only (`--url`, `--text`, `find role|text|label`).
  Never use the AI `chat` command.
- Flows read **seeded data only** (`acme/payments-api`, PR #482, the built-in agents)
  and must not trigger a model call or write data other flows depend on.
- Use `{BASE}` in URLs; never hardcode a host or port.
- Number new specs after the last one (`08-…`); a flow's position may depend on the
  ones before it.

## Gotchas

- Flows 02/04/05 assume the seeded repo is the **only** repo. Against a normal dev
  DB they land on the wrong repo and fail. Use the hermetic runner.
- Visible text in assertions comes from `client/messages/en/*.json`; renaming a
  string there can break a flow.
- Never `docker compose down -v` to reset; it wipes the dev DB volume.

## Read when

- **Read `README.md`** (this folder) when you write a new flow or debug a failing one.
- **Read `../TESTING.md`** when you change what the e2e suite covers or its CI job.
- **Read `specs/`** before building a feature here; write a spec first for a new
  lesson feature (template: `../docs/spec-template.md`). The runner only loads
  `*.flow.json`, so Markdown specs can live next to the flows.
- **Read `docs/`** (this folder) when you need design notes beyond the README.
- **Read `INSIGHTS.md`** before non-trivial changes here; append new findings to it.
