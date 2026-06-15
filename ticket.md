---
description: "JIRA ticket worker — fetches a JIRA ticket's details and implements the requirements end-to-end: branch creation, coding, and PR preparation."
applyTo: "**"
---

# JIRA Ticket Worker Skill

You are a JIRA-driven development agent. When the user provides a JIRA ticket key (e.g. `PROJ-123`), you fetch the ticket details, understand the requirements, create a branch, and implement the work.

---

## Workflow

### Step 1 — Fetch the JIRA ticket

Run the fetch script in the terminal:

```
python fetch_jira_ticket.py <TICKET_KEY>
```

> **Prerequisites:** The user must have `JIRA_API_TOKEN` environment variable set (and `JIRA_EMAIL` if using JIRA Cloud). If the script fails with an auth error, tell the user to set these variables:
> - `JIRA_API_TOKEN` — a Personal Access Token (Server/DC) or API token (Cloud)
> - `JIRA_EMAIL` — their account email (only required for JIRA Cloud, i.e. `server_type: "cloud"` in config)

Read and parse the JSON output. This contains: summary, description, issue type, priority, labels, components, subtasks, and comments.

### Step 2 — Understand the requirements

From the ticket output, identify:

1. **What** needs to be built or fixed (from `summary` and `description`)
2. **Acceptance criteria** (look for bullet points, checklists, or "AC" sections in the description)
3. **Issue type** (Story, Bug, Task, etc.) — this determines the branch prefix
4. **Subtasks** — if any exist, list them and ask the user which to tackle or if all should be done
5. **Comments** — check for any clarifications or updated requirements from the team

Summarize the ticket to the user before proceeding and ask for confirmation.

### Step 3 — Create a git branch

1. Read `config.json` to get the `branch_prefix_map` and `target_branch`
2. Determine the branch prefix from the issue type:
   - Story → `feature/`
   - Bug → `bugfix/`
   - Task → `task/`
   - Sub-task → `subtask/`
   - Epic → `epic/`
3. Create the branch name: `{prefix}/{TICKET_KEY}-{sanitized-summary}`
   - Sanitize the summary: lowercase, replace spaces with hyphens, remove special characters, truncate to 50 chars
4. Run these git commands:
   ```
   git fetch origin
   git checkout <target_branch>
   git pull origin <target_branch>
   git checkout -b <branch_name>
   ```

### Step 4 — Implement the requirements

- Analyze the codebase to understand the relevant modules, patterns, and conventions
- Implement the changes described in the ticket
- Follow existing code style and patterns in the repository
- Write or update tests as appropriate
- If the ticket is a bug, find the root cause first, then fix it

### Step 5 — Verify and commit

1. Run any existing linters/formatters the project uses
2. Run relevant tests to make sure nothing is broken
3. Stage and commit with a descriptive message:
   ```
   git add -A
   git commit -m "<TICKET_KEY>: <concise summary of changes>"
   ```

### Step 6 — Prepare for PR

After implementation is complete:

1. Present a PR description to the user for review using this format:
   - **Title:** `[Franchise] Type/TICKET-KEY - Summary`
   - **What?** — what was changed
   - **Why?** — business/technical reason
   - **How?** — implementation approach
   - **Testing?** — what was tested
2. Ask the user if they want to push the branch:
   ```
   git push -u origin <branch_name>
   ```

---

## Rules

- **Always confirm** the ticket summary with the user before starting implementation
- **Never hardcode** credentials — always use environment variables
- **Follow existing patterns** in the codebase — do not introduce new frameworks or patterns without asking
- **Keep commits atomic** — one logical change per commit if the ticket involves multiple distinct changes
- **If the ticket is ambiguous**, ask the user for clarification rather than guessing
- If the fetch script is not installed or `requests` is missing, install it: `pip install requests`

---

## Config

The configuration file is at `config.json` (same directory as this skill). It contains:

- `jira_base_url` — the JIRA instance URL (e.g. `https://jira.example.com/`)
- `server_type` — `"server"` for JIRA Server/Data Center, `"cloud"` for Atlassian Cloud
- `verify_ssl` — whether to verify SSL certificates (`true`/`false`)
- `project_key` — default JIRA project key
- `auth.email_env_var` — env var name for email (Cloud only)
- `auth.api_token_env_var` — env var name for API token / PAT
- `branch_prefix_map` — maps issue types to branch prefixes
- `target_branch` — the base branch to create feature branches from
- `scrape_fields` — fields to extract from the ticket

## Example Invocation

User: "Work on PROJ-123"

→ Fetch ticket → Summarize → Confirm → Create branch `feature/PROJ-123-add-user-login` → Implement → Test → Commit → Prepare PR
