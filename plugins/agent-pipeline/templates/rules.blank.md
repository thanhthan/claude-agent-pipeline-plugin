# Pipeline rules — <project name>

This file holds the project-SPECIFIC rules. Every agent in the pipeline (planner, coder, reviewer, tester, and the ticket-integration agents) reads it. Rules here OVERRIDE agent defaults on conflict.

Fill in the sections below (delete sections you don't need):

## Architecture / conventions

- Style guide: (e.g. eslint airbnb / prettier default / dotnet-format)
- Required architecture pattern: (e.g. layered, ports-adapters, feature-folder)
- Files / directories that must NOT be modified: (e.g. generated code, vendored third-party)

## Test

- Framework: (e.g. vitest / xunit / jest / pytest)
- Suite command: (e.g. `pnpm test`, `dotnet test src/Foo.slnx`)
- Coverage target: (e.g. 80% on new code)
- Mocking rules: (e.g. "no DB mocks", "use in-memory sqlite")

## Quality gate before the Coder moves a task to review

- (e.g. `pnpm typecheck && pnpm lint`)
- (e.g. `dotnet build --no-restore -warnaserror`)

## Review checklist (extra for the reviewer)

- (e.g. unique list keys, null-guard for enums, aria-busy on async regions)
- (e.g. no hardcoded secrets, no SQL injection, EF query filter uses correct scope)

## Security / compliance

- (e.g. PHI/PII fields must be masked via `[Phi(Ssn)]`)
- (e.g. explicit authorization stance required on every endpoint)

## Ticket / commit conventions (if any)

- Branch naming: (e.g. `feature/us-<n>-<slug>`)
- Commit message: (e.g. conventional commits, no attribution)
- PR: (e.g. one PR per task, open draft before QA handoff)

## Other notes

(any context the agents need to know that they cannot infer from code)
