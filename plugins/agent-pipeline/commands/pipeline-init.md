---
description: Initialize agent-pipeline in the current project — create pipeline/tasks/*, .claude/pipeline.rules.md from a template, and (optionally) Jira/ADO/GitHub config files. Run once per project.
---

Init configuration: $ARGUMENTS

If $ARGUMENTS contains any of the keywords `jira`, `ado`, `github` (lowercase, case-insensitive), also copy the matching ticket config. Examples:
- `/pipeline-init` — base only
- `/pipeline-init jira` — base + Jira config
- `/pipeline-init ado` — base + Azure DevOps config
- `/pipeline-init github` — base + GitHub config
- `/pipeline-init jira ado` — base + both Jira and ADO

Execute in this exact order:

1. **Create the pipeline directory (required)**:
   ```bash
   mkdir -p pipeline/tasks/pending pipeline/tasks/review pipeline/tasks/testing pipeline/tasks/done
   touch pipeline/tasks/pending/.gitkeep pipeline/tasks/review/.gitkeep pipeline/tasks/testing/.gitkeep pipeline/tasks/done/.gitkeep
   ```

2. **Create `.claude/pipeline.rules.md`** — if the file does NOT already exist, ASK the user to pick a template:
   - `blank` — empty scaffold, fill in yourself
   - `generic-react` — React/TypeScript project (vitest, eslint, 80% coverage)
   - `generic-dotnet` — .NET project (xunit, dotnet build, 80% coverage)
   - `surency` — Surency Admin Hub (auth stance, employer scoping, PHI masking, CodeRabbit checklist)

   Once the user picks, copy the corresponding template from `${CLAUDE_PLUGIN_ROOT}/templates/rules.<choice>.md` to `.claude/pipeline.rules.md`.
   If the file already exists, DO NOT overwrite — report to the user and ask whether they want to view another template for manual merging.

3. **Copy ticket configs (only when $ARGUMENTS requests it)**:

   - If `jira` → `mkdir -p .claude/pipeline && cp ${CLAUDE_PLUGIN_ROOT}/templates/jira.config.md .claude/pipeline/jira.config.md` (only if the file does not exist). After copying, print INSTRUCTIONS to fill in the square-bracket placeholders and install the Jira MCP: `claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2` then `/mcp` to OAuth.

   - If `ado` → `mkdir -p .claude/pipeline && cp ${CLAUDE_PLUGIN_ROOT}/templates/ado.config.md .claude/pipeline/ado.config.md`. Instruct the user to fill it in and confirm the `azure-devops` MCP server is connected.

   - If `github` → `mkdir -p .claude/pipeline && cp ${CLAUDE_PLUGIN_ROOT}/templates/github.config.md .claude/pipeline/github.config.md`. Instruct the user to fill it in and confirm `gh auth login` is done.

4. **Suggest .gitignore updates** — ASK the user whether to add `pipeline/tasks/*/` to `.gitignore`. Keep committed if the team wants to track pipeline state via git; ignore otherwise.

5. **Final report**: list files created, config files that still need filling in, and next-step suggestions (`/pipeline "..."` or `/jira-pipeline` / `/ado-pipeline` / `/github-pipeline`).

Apply the `human-ticket-voice` skill when writing the final report.
