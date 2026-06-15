# 🎫 TICKET.MD

A Claude skill that fetches JIRA ticket details and drives end-to-end development — from branch creation through implementation to PR preparation.

Works with both **JIRA Server / Data Center** and **Atlassian Cloud**.

---

## 📁 Files

| File | Purpose |
|---|---|
| `ticket.md` | Claude skill definition (agent instructions) |
| `fetch_jira_ticket.py` | Python script to fetch ticket details via JIRA REST API |
| `config.json` | Configuration — JIRA URL, auth, branch naming, etc. |

---

## ⚡ Quick Start

### 1. Install dependencies

```bash
pip install requests
```

### 2. Configure `config.json`

Open the file and replace the placeholders:

```jsonc
{
    "jira_base_url": "https://jira.yourcompany.com/",   // ← your JIRA instance URL
    "jira_board_url": "https://jira.yourcompany.com/browse/",
    "project_key": "MYPROJ",                             // ← your project key
    "server_type": "server",                              // ← "server" or "cloud"
    "verify_ssl": false,                                  // ← set true if your cert is trusted
    ...
}
```

> **`server_type`** determines the auth method and API version automatically:
>
> | | JIRA Server / Data Center | Atlassian Cloud |
> |---|---|---|
> | `server_type` | `"server"` | `"cloud"` |
> | Auth | Bearer token (PAT) | Basic auth (email + API token) |
> | API version | REST API v2 | REST API v3 |

### 3. Set environment variables

**JIRA Server / Data Center** — only the token is needed:

```powershell
# PowerShell
$env:JIRA_API_TOKEN = "<your-personal-access-token>"
```

```bash
# Bash / Zsh
export JIRA_API_TOKEN="<your-personal-access-token>"
```

**Atlassian Cloud** — both email and token are needed:

```powershell
$env:JIRA_EMAIL = "you@company.com"
$env:JIRA_API_TOKEN = "<your-api-token>"
```

> **Where to generate tokens:**
> - **Server/DC** — Your JIRA profile → Personal Access Tokens
> - **Cloud** — https://id.atlassian.com/manage-profile/security/api-tokens

### 4. Verify the setup

```bash
python fetch_jira_ticket.py MYPROJ-123
```

You should see a JSON output with the ticket's summary, description, status, comments, etc.

---

## 🚀 Usage

### With Copilot (recommended)

Simply ask Copilot to work on a ticket:

```
Work on PROJ-456
```

The skill will automatically:
1. Fetch the ticket details
2. Summarize requirements and ask for confirmation
3. Create a git branch (e.g. `feature/PROJ-456-add-user-login`)
4. Implement the changes
5. Run tests and commit
6. Prepare a PR description

### Standalone (fetch only)

```bash
# Parsed output (default)
python fetch_jira_ticket.py PROJ-456

# Raw API response
python fetch_jira_ticket.py PROJ-456 --raw

# Custom config path
python fetch_jira_ticket.py PROJ-456 --config /path/to/config.json
```

---

## ⚙️ Configuration Reference

All settings live in `config.json`:

| Field | Description | Example |
|---|---|---|
| `jira_base_url` | Root URL of your JIRA instance | `https://jira.example.com/` |
| `jira_board_url` | Browse URL for linking tickets | `https://jira.example.com/browse/` |
| `project_key` | Default JIRA project key | `MYPROJ` |
| `server_type` | `"server"` (Server/DC) or `"cloud"` (Atlassian Cloud) | `"server"` |
| `verify_ssl` | Verify SSL certificates | `true` / `false` |
| `auth.email_env_var` | Env var name for email (Cloud only) | `"JIRA_EMAIL"` |
| `auth.api_token_env_var` | Env var name for token/PAT | `"JIRA_API_TOKEN"` |
| `branch_prefix_map` | Maps issue types to branch prefixes | `{"Story": "feature", ...}` |
| `target_branch` | Base branch for new feature branches | `"main"` |
| `scrape_fields` | Fields to extract from the ticket | `["summary", "description", ...]` |

---

## 🌿 Branch Naming

Branches are auto-created based on the issue type:

| Issue Type | Prefix | Example |
|---|---|---|
| Story | `feature/` | `feature/PROJ-123-add-user-login` |
| Bug | `bugfix/` | `bugfix/PROJ-456-fix-null-pointer` |
| Task | `task/` | `task/PROJ-789-update-dependencies` |
| Sub-task | `subtask/` | `subtask/PROJ-101-add-unit-tests` |
| Epic | `epic/` | `epic/PROJ-200-user-management` |

---

## 🔧 Troubleshooting

| Problem | Solution |
|---|---|
| `Authentication failed` | Verify your `JIRA_API_TOKEN` is valid and not expired. Regenerate if needed. |
| `Ticket not found` | Check the ticket key and make sure `jira_base_url` is correct. |
| `SSL certificate error` | Set `"verify_ssl": false` in config (common for corporate instances with internal CAs). |
| `requests not found` | Run `pip install requests`. |
| `Config file not found` | Ensure `config.json` is in the same directory as the script, or use `--config`. |
