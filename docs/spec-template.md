# Spec template

Shared template for every module's `specs/` folder. A spec is written **before**
implementation and kept in sync while the feature is built. It says what to build
and how we'll know it works; it doesn't restate architecture (link the READMEs).

## Where a spec lives

In `<module>/specs/` of the module that owns most of the change. Other modules
the feature touches get a one-line pointer file or a link from their own spec.

## Naming

`<lesson>-<slug>.md`, for example `L01-run-cost-badge.md`. Use `x-<slug>.md` for
work outside the course lessons.

## Template

```markdown
# <Feature name>

Status: draft | in progress | done
Lesson: L0X
Modules: server, client, reviewer-core, e2e

## Problem
What the user can't do today, in one or two sentences.

## Behaviour
What the user sees and does, step by step.

## Contracts
New or changed routes, Zod schemas (both shared copies), DB tables/columns.

## Out of scope
What this spec deliberately doesn't cover.

## Acceptance
- [ ] Checks a reviewer can run, including the test file names that cover them.
```
