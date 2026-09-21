# Azure DevOps config for pipeline (fill in before running /ado-pipeline)

ADO_ORG: [e.g. yourorg]                                  # org slug in dev.azure.com/<org>
ADO_PROJECT: [e.g. MyProduct]                            # project name
TEAM: [optional — leave blank for single-team setups]
ASSIGNEE_EMAIL: @me                                      # keep as-is to scan work items assigned to the current identity
TESTER_EMAIL: [tester email, e.g. tester@yourcompany.com]
READY_FOR_QA_STATE: [EXACT State name in the workflow, e.g. "Ready for Test" / "In QA" / "Resolved"]

# Setup

1. Install the Azure DevOps MCP (if not already installed):
   - See https://github.com/microsoft/azure-devops-mcp
   - Or `claude mcp add azure-devops <command>` with the corresponding server.
2. Confirm that `wit_*` tools appear when you run `/mcp` in Claude Code.
3. The OAuth account must have the following permissions:
   - Read work items in the project
   - Update work item State + AssignedTo
   - Comment on work items

# Notes

- READY_FOR_QA_STATE must EXACTLY match the State name in the workflow (case-insensitive match). Not every transition is valid — if handoff reports "invalid transition", check the workflow of the work item type (User Story / Bug / Task).
- ASSIGNEE_EMAIL = `@me` is interpreted by ADO WIQL as the OAuth identity. To scan on behalf of someone else, replace with the full email.
