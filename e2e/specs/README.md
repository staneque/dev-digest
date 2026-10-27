# e2e specs

This folder holds two kinds of files:

- `NN-name.flow.json`: the executable browser flows. `run.ts` loads only files
  ending in `.flow.json`.
- `*.md`: feature specs owned by `e2e/` (new suites, runner changes), written
  before the code. Template, naming and placement rules:
  [`../../docs/spec-template.md`](../../docs/spec-template.md).
