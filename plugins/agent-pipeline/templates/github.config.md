# GitHub Issues config for pipeline (fill in before running /github-pipeline)

GH_REPO: [owner/repo, e.g. myorg/myapp]
ASSIGNEE: @me                                        # keep as-is to scan issues assigned to the current gh user
TESTER_HANDLE: [github username of the tester, e.g. jdoe — NOT the email]
READY_FOR_QA_LABEL: [label name added at handoff, e.g. "ready-for-qa" — must exist on the repo]

# Setup

1. Install the GitHub CLI:
   - macOS: `brew install gh`
   - Others: https://cli.github.com/
2. Authenticate: `gh auth login` (select "GitHub.com", HTTPS, authenticate via web browser).
3. Verify: `gh auth status` should show your user with `repo` scope.
4. Ensure the `READY_FOR_QA_LABEL` exists on the repo:
   ```
   gh label list --repo <owner/repo>
   ```
   If missing, create it:
   ```
   gh label create "ready-for-qa" --color FBCA04 --description "Ready for QA verification" --repo <owner/repo>
   ```

# Notes

- The account you authenticate with must have write access to the repo (to add labels, assignees, comments). Public-repo-only auth cannot mutate issues on other repos.
- `TESTER_HANDLE` is the GitHub username WITHOUT the `@` prefix.
- If the repo uses GitHub Projects (v2) with a status field, this plugin only manipulates labels + assignees; move-to-column has to be done manually or via a follow-up automation.
