# Warp Migration Agent Plugin

An Agentic App plugin for Packfiles Warp migration workflows. It helps teams plan migrations, diagnose failed/stuck migrations, and validate Migration HQ configuration for Azure DevOps and Bitbucket Server to GitHub migrations.

## Plugin Structure

```
.
├── plugin.json
├── .mcp.json
├── agents/
│   └── main.agent.md
└── skills/
    ├── diagnose-migration/
    │   ├── SKILL.md
    │   └── failure-catalog.md
    ├── plan-migration/
    │   ├── SKILL.md
    │   ├── output-template.md
    │   └── scoring.md
    └── validate-config/
        ├── SKILL.md
        └── warp-yml-schema.md
```

Agents live in `agents/` and skills in `skills/<name>/` at the repository
root, as required by the Copilot CLI plugin spec — the harness discovers the
**main agent** (`agents/main.agent.md`) from these paths declared in
[`plugin.json`](plugin.json).

---

## Agent and Skills

### Main Agent

`agents/main.agent.md` defines the behavior and orchestration for:

- Diagnosing migration failures on backlog issues
- Planning wave-based migrations
- Validating `config/warp.yml` updates

### diagnose-migration

Folder: `skills/diagnose-migration`
- Main instructions: `SKILL.md`
- Classification reference: `failure-catalog.md`
- Purpose: classify failed/stuck migration issues from issue body, labels, and comments
- Output behavior: post a diagnosis comment to the issue with root cause, remediation, and retry command (typically `/migrate`)

### plan-migration

Folder: `skills/plan-migration`

- Main instructions: `SKILL.md`
- Scoring rules: `scoring.md`
- Document format: `output-template.md`
- Purpose: score repositories, assign migration waves, apply `planning:*` labels, and generate a customer-ready migration strategy document
- Output behavior: generates `migration-strategy.md` (default location: repository root unless the user specifies otherwise)

### validate-config

Folder: `skills/validate-config`

- Main instructions: `SKILL.md`
- Schema reference: `warp-yml-schema.md`
- Purpose: validate `config/warp.yml` for schema correctness, policy conflicts, and security implications
- Output behavior:
  - Review mode: post a structured findings comment
  - Authoring mode: return corrected YAML snippets in chat

---

## Deployment

### Local Testing (Copilot CLI)

```bash
# Install the plugin from the repo root
copilot plugin install .

# Verify
copilot plugin list

# Interactive session
copilot
/agent              # confirm packfiles-migration-agent loaded
/skills list        # confirm skills loaded
```

### Deploy as an Agentic App

1. **Make the repository public.** The Agent HQ platform requires the plugin
   repository to be public. Flip visibility from the repository's **Settings →
   General → Danger Zone → Change visibility**. Do not stage files blindly with
   `git add .` — see [`.gitignore`](.gitignore), which keeps internal reference
   material (e.g. PDFs) and credential files out of version control.

2. **Register a GitHub App** at GitHub Settings → Developer settings → GitHub Apps:
   - Permissions (least privilege): Issues (read/write), Pull Requests
     (read/write), Contents (read/write). Contents **write** is required because
     `plan-migration` writes `migration-strategy.md` and `validate-config` can
     update `config/warp.yml`. To keep the agent from committing to Migration HQ,
     scope Contents to read-only and have skills return output in chat instead.
   - Subscribe to events: `issues`, `issue_comment`, `pull_request`, `pull_request_review_comment`
   - Install the App only on the organizations/repositories that use Warp — not
     on all repositories.

3. **Configure as Agentic App** (type=plugin), pointing to the public repo.

4. **Install the App** on the organization(s) using Warp for migrations.

### Authentication (OIDC)

The plugin uses OIDC to authenticate with the Warp API — no long-lived secrets required. The platform:
1. Signs a short-lived JWT with claims identifying the agent, org, and repo.
2. Exchanges it at `https://warp.packfiles.io/token` for a Warp access token.
3. Injects the token into the MCP server automatically.
4. Revokes the token when the job completes.

---

## Extending

- Add more skills (e.g., `generate-report`, `bulk-retry`, `audit-permissions`)
- Add hooks for deterministic pre-processing (e.g., always validate credentials before `/migrate`)
- Add additional agents for specialized workflows (e.g., a `post-migration-agent` that sets up branch protection and CODEOWNERS after migration)

## Security

This repository is public because the contents are synced and run as a deployed
Agentic App. Every file here defines production agent behavior, and the agent
processes untrusted issue/PR/comment content. The plugin stores **no secrets** —
it authenticates to the Warp API via OIDC. See [`SECURITY.md`](SECURITY.md) for
the security model and how to report a vulnerability, and
[`.github/CODEOWNERS`](.github/CODEOWNERS) for required review on agent changes.

## License

MIT
