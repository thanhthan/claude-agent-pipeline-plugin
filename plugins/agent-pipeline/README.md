# agent-pipeline (plugin)

Installable Claude Code plugin providing a four-agent auto-task pipeline (`planner` → `coder` → `reviewer` → `tester`) with optional Jira / Azure DevOps / GitHub Issues ingestion. Configurable per project via `.claude/pipeline.rules.md`.

See the [top-level README](../../README.md) for install and usage instructions.

## Layout

- `agents/` — subagent definitions (base four + six ticket-integration agents).
- `commands/` — slash commands: `/pipeline`, `/jira-pipeline`, `/ado-pipeline`, `/github-pipeline`, `/pipeline-init`.
- `skills/human-ticket-voice/` — writing-style checklist applied whenever an agent writes text that goes to a human (ticket comments, PR bodies, QA handoff notes).
- `templates/` — starter files copied into the project by `/pipeline-init`.
