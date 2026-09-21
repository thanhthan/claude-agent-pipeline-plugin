# Jira config for pipeline (fill in before running /jira-pipeline)

JIRA_SITE: [e.g. yourcompany.atlassian.net]
PROJECT_KEY: [e.g. ABC]
ASSIGNEE_JQL: currentUser()  # keep as-is to scan tickets assigned to the current OAuth account
TESTER_ACCOUNT: [tester's Jira email, e.g. tester@yourcompany.com]
READY_FOR_QA_STATUS: [EXACT status name in the Jira workflow, e.g. "Ready for QA" — open any ticket in the project and check the status column / transition button for the exact name]

# Setup

1. Install the Jira MCP:
   ```
   claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2
   ```
2. In Claude Code, run `/mcp` to OAuth with your Jira Cloud instance.
3. Jira Cloud only — for self-hosted Server / Data Center, use a different MCP server (`sooperset/mcp-atlassian`).

# Notes

- READY_FOR_QA_STATUS must EXACTLY match the transition name in the workflow (case-insensitive). Every project may name it differently: "QA Ready", "Ready for Test", "In QA", etc.
- If you're unsure, run `/jira-pipeline` once, then ask Claude: "list available transitions for ticket <KEY>" to get the exact name.
- The tester can be a service account (no need to be a real person) as long as the Jira account is assignable.
