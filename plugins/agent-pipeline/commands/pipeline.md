---
description: Run the full pipeline planner → coder → reviewer → tester for one request, looping until every task lands in pipeline/tasks/done/.
---

Request to process: $ARGUMENTS

BEFORE starting:
- Check `.claude/pipeline.rules.md` — if it exists, read it and remember the constraints that apply to this pipeline.
- Check `pipeline/tasks/{pending,review,testing,done}/` — if they don't exist, suggest the user run `/pipeline-init` first.

Execute in this exact order, no skipping:

1. Invoke the `planner` subagent with the request above to create tasks in `pipeline/tasks/pending/`.
2. Loop the following until `pipeline/tasks/pending/`, `pipeline/tasks/review/`, and `pipeline/tasks/testing/` are ALL empty:
   a. If `pipeline/tasks/pending/` has an eligible task (no unresolved dependencies) → invoke the `coder` subagent.
   b. If `pipeline/tasks/review/` has a task → invoke the `reviewer` subagent.
   c. If `pipeline/tasks/testing/` has a task → invoke the `tester` subagent.
   d. If a full round (a, b, c) changes no state at all → stop and report a deadlock, listing the stuck tasks and why.
3. When every task is in `pipeline/tasks/done/`, print a summary: completed tasks, total number of Coder rework loops (if countable), and suggested next steps (e.g. run build, commit, open PR). Apply the `human-ticket-voice` skill to this summary.

Print a short status line after each step (which task moved from where to where) so the user can follow progress in real time.
