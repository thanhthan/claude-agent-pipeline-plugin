---
description: Scan Azure DevOps for exactly one work item assigned to you, auto code/review/test it, then transition its State to ready-for-QA and assign the tester.
---

BEFORE starting:
- Check `.claude/pipeline/ado.config.md` exists and is filled in. If not, stop and suggest `/pipeline-init ado`.
- Check `.claude/pipeline.rules.md` — if it exists, read it and remember.
- Confirm the `azure-devops` MCP server is connected (otherwise `wit_*` tools won't be available).

Execute in this exact order, no skipping, no parallel work items:

1. Invoke the `ado-scanner` subagent.
   - If it reports "a ticket is already in progress" or "no work items waiting" → stop and report the reason verbatim.
2. If ado-scanner picked one work item, loop until it reaches `done/`:
   a. `pending/` → `coder`
   b. `review/` → `reviewer`
   c. `testing/` → `tester`
   d. Full round with no state change → stop, deadlock with reason.
3. When the task lands in `done/`, invoke the `ado-handoff` subagent to change the State and assign the tester.
4. Print a summary: ADO id, what was done, test results, current State, assignee. Apply the `human-ticket-voice` skill.

Print a short status line after each step.
