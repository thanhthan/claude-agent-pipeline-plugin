# Pipeline rules — Generic .NET / C# project

## Architecture / conventions

- Style: `dotnet format` (StyleCop / EditorConfig at repo root).
- Pattern: layered (Domain / Application / Infrastructure / HttpApi) — no layer-skipping.
- Files that must NOT be modified: `bin/`, `obj/`, committed migration files (add a new migration instead of editing an old one), generated clients.

## Test

- Framework: **xUnit** with **NSubstitute** and **Shouldly** (or **Moq** + **FluentAssertions** — follow the project).
- Suite command: `dotnet test <SolutionFile>` (e.g. `dotnet test src/App.slnx`).
- Coverage target: **80% on new code** (SonarQube new-code gate if in use).
- Test naming: `MethodName_Scenario_ExpectedBehavior`.
- Tests instantiate the SUT directly (avoid ABP TestBase unless required).

## Quality gate before the Coder moves a task to review

```bash
dotnet build <SolutionFile> --no-restore -warnaserror
dotnet test <SolutionFile> --no-build
```
Zero warnings and zero test failures.

## Review checklist (extra)

- No hardcoded connection strings or secrets.
- No SQL injection (use parameterized queries or EF LINQ, no string concatenation).
- Correct async: `await` every Task, no `.Result` / `.Wait()`.
- Null-guard parameters on `public` methods.
- Meaningful exceptions (`UserFriendlyException` for user-facing, custom exceptions for domain).
- CA1859 / CA1860 and other CA analyzers: fix rather than suppress.

## Security

- Explicit authorization stance on every endpoint (`[Authorize]` / `[AllowAnonymous]`).
- Logs never leak PII/PHI (mask via attribute when available).

## Ticket / commit

- Branch: `feature/<n>-<slug>` or `fix/<slug>`.
- Commit: conventional commits, no AI attribution.
- Migration: create the EF migration in the same commit as the entity/DbContext change.
