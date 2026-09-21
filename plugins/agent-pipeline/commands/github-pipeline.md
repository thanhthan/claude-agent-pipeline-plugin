---
description: Scan GitHub Issues for exactly one issue assigned to you, auto code/review/test it, then add the ready-for-qa label and assign the tester.
---

BEFORE starting:
- Check `.claude/pipeline/github.config.md` exists and is filled in. If not, stop and suggest `/pipeline-init github`.
- Check `.claude/pipeline.rules.md` — if it exists, read it and remember.
- Confirm `gh --version` works and `gh auth status` is OK.

Execute in this exact order:

1. Invoke the `github-scanner` subagent.
   - If it reports "an issue is already in progress" or "no issues waiting" → stop and report verbatim.
2. If github-scanner picked one issue, loop:
   a. `pending/` → `coder`
   b. `review/` → `reviewer`
   c. `testing/` → `tester`
   d. Full round with no state change → stop, deadlock.
3. Task in `done/` → invoke the `github-handoff` subagent.
4. Print a summary: issue number, what was done, test results, label added, tester assigned. Apply the `human-ticket-voice` skill.

Print a short status line after each step.
