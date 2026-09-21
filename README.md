# claude-agent-pipeline-plugin

Multi-agent auto-task pipeline for **Claude Code**, packaged as an installable plugin.

Four coordinated subagents — `planner` → `coder` → `reviewer` → `tester` — pass work through `pipeline/tasks/{pending,review,testing,done}/` on disk. Optional ticket-integration agents pull work directly from **Jira Cloud**, **Azure DevOps Boards**, or **GitHub Issues** and hand results back at the end.

Every project can override the base agents' behavior with a single file: `.claude/pipeline.rules.md`. Ship-ready templates are included for React/TypeScript, .NET, and an enterprise ABP monorepo preset.

---

## Install

### 1. Add this repo as a marketplace

Inside any project where you use Claude Code, run:

```
/plugin marketplace add https://github.com/thanhthan/claude-agent-pipeline-plugin
```

### 2. Install the plugin

```
/plugin install agent-pipeline@thanh-plugins
```

Restart Claude Code (or reload the plugin) to make the new agents and commands available.

### 3. Initialize the pipeline in the current project

Run once per project:

```
/pipeline-init              # base only (no ticket integration)
/pipeline-init jira         # base + Jira config
/pipeline-init ado          # base + Azure DevOps config
/pipeline-init github       # base + GitHub Issues config
/pipeline-init jira ado     # combine multiple sources
```

The command creates `pipeline/tasks/{pending,review,testing,done}/`, asks you which rules template to seed `.claude/pipeline.rules.md` from (blank / generic-react / generic-dotnet / enterprise-abp), and copies the ticket-integration config into `.claude/pipeline/<source>.config.md` when requested.

---

## Usage

### Local pipeline (no ticket source)

Automate the whole flow for one request:

```
/pipeline Add email + password login with validation and rate limiting
```

Claude Code will:
1. Invoke `planner` to break the request into tasks under `pipeline/tasks/pending/`.
2. Loop `coder` → `reviewer` → `tester` for each task until all reach `pipeline/tasks/done/`.
3. Print a final summary.

To step through manually instead of auto-running, invoke each subagent by name:

```
@planner Add email + password login
@coder
@reviewer
@tester
```

### Jira / ADO / GitHub pipeline

Fill in the corresponding `.claude/pipeline/<source>.config.md` first, then:

```
/jira-pipeline
/ado-pipeline
/github-pipeline
```

Each run picks EXACTLY ONE ticket assigned to you (priority-then-oldest ordering), works it through the internal pipeline, and at the end transitions the ticket to your configured ready-for-QA state and assigns the tester. Only one ticket in flight at a time — the hand-off agent refuses to start a new one until the current one is fully done.

---

## Project-specific rules — `.claude/pipeline.rules.md`

Every agent reads this file first (if it exists) and its rules OVERRIDE the agent defaults. Use it to encode:

- Style guide, mandatory architecture pattern, files that must not be modified.
- Test framework and suite command, coverage target, mocking rules.
- Quality gate the Coder must pass before moving to review (typecheck, lint, build).
- Extra review checklist (recurring bot findings, security must-haves).
- Ticket / commit / branch conventions.

Bundled templates in `plugins/agent-pipeline/templates/`:

| Template            | Fits                                                                 |
|---------------------|----------------------------------------------------------------------|
| `rules.blank`       | Empty scaffold to fill in yourself.                                  |
| `rules.generic-react` | React/TypeScript + vitest + eslint + 80% coverage.                 |
| `rules.generic-dotnet` | .NET + xUnit + `dotnet build --warnaserror` + 80% coverage.       |
| `rules.enterprise-abp` | Enterprise ABP monorepo (React + .NET): strict auth stance, tenant scoping, PHI masking, automated-review checklist. |

---

## What's in the plugin

```
plugins/agent-pipeline/
├── .claude-plugin/plugin.json
├── agents/
│   ├── planner.md       coder.md         reviewer.md      tester.md
│   ├── jira-scanner.md  jira-handoff.md
│   ├── ado-scanner.md   ado-handoff.md
│   └── github-scanner.md github-handoff.md
├── commands/
│   ├── pipeline.md         pipeline-init.md
│   ├── jira-pipeline.md    ado-pipeline.md    github-pipeline.md
├── skills/
│   └── human-ticket-voice/SKILL.md   # anti-AI-tell writing checklist for public comments
└── templates/
    ├── rules.blank.md         rules.generic-react.md
    ├── rules.generic-dotnet.md rules.enterprise-abp.md
    ├── jira.config.md         ado.config.md      github.config.md
```

### Integrations with other plugins

- **[Superpowers](https://github.com/obra/superpowers)** — if installed, agents wire into `brainstorming`, `writing-plans`, `test-driven-development`, `requesting-code-review`, `systematic-debugging`, `verification-before-completion`, `finishing-a-development-branch`. If not installed, agents ignore the missing skills and still work with their built-in logic.
- **[graphify](https://github.com/Graphify-Labs/graphify)** — if `graphify-out/graph.json` exists in the project, agents prefer `graphify query`/`graphify affected` over wide Grep/Read to save tokens. Falls back to Grep/Glob when unavailable.

### Ticket-integration prerequisites

| Source    | Requires                                                                             |
|-----------|--------------------------------------------------------------------------------------|
| Jira      | Atlassian MCP: `claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2`, then `/mcp` to OAuth. Jira Cloud only. |
| Azure DevOps | `azure-devops` MCP server connected (e.g. [microsoft/azure-devops-mcp](https://github.com/microsoft/azure-devops-mcp)). |
| GitHub    | `gh` CLI installed (`brew install gh`) and authenticated (`gh auth login`).          |

---

## Notes / caveats

- **Do NOT run the pipeline fully unattended for critical work.** It's fine for small, well-scoped tasks; still read the final summary and the diff before committing/deploying.
- **Models per agent**: all agents default to `sonnet`. Edit the `model:` field in each agent's frontmatter to route Planner/Reviewer to a stronger model (e.g. `opus`) for complex domains while keeping Coder/Tester on `sonnet` for cost.
- **Tool scoping**: to prevent Coder/Tester from running arbitrary Bash, drop `Bash` from that agent's `tools:` list; it will then only propose commands for you to run.
- **Deadlock detection**: if a task keeps bouncing between Reviewer/Tester and pending, the pipeline reports the stuck task; there is no built-in max-retry counter — add one in the agent file if you want stricter behavior.
- **One ticket at a time**: the ticket-integration commands enforce this. Finish (or manually resolve) the in-flight ticket before starting another `/jira-pipeline` / `/ado-pipeline` / `/github-pipeline`.

---

## License

MIT — see [LICENSE](LICENSE).
