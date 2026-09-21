---
description: Scan Jira for exactly one ticket assigned to you, auto code/review/test it, then transition to Ready for QA and assign the tester.
---

BEFORE starting:
- Check `.claude/pipeline/jira.config.md` exists and is filled in (no square-bracket placeholders left). If not, stop and suggest the user run `/pipeline-init jira`.
- Check `.claude/pipeline.rules.md` — if it exists, read it and remember.

Execute in this exact order, no skipping, no parallel tickets:

1. Invoke the `jira-scanner` subagent.
   - If it reports "a ticket is already in progress" or "no tickets waiting" → stop the whole flow and report the reason verbatim.
2. If jira-scanner picked one ticket (created a file in `pipeline/tasks/pending/`), loop until it reaches `pipeline/tasks/done/`:
   a. Task in `pending/` → invoke `coder`
   b. Task in `review/` → invoke `reviewer`
   c. Task in `testing/` → invoke `tester`
   d. A full round with no state change → stop, report deadlock with reason.
3. When the task lands in `done/`, invoke the `jira-handoff` subagent to transition the Jira ticket to Ready for QA and assign the tester.
4. Print a summary: ticket key, what was done, test results, current Jira status, assignee. Apply the `human-ticket-voice` skill to this summary.

Print a short status line after each step so the user can follow along in real time.
