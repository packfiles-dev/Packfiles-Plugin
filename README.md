# Warp Migration Agent Plugin

An Agentic App plugin for Packfiles Warp migration workflows. It helps teams plan migrations, diagnose failed/stuck migrations, and validate Migration HQ configuration for Azure DevOps and Bitbucket Server to GitHub migrations.

## Plugin Structure

```
.
├── plugin.json
├── .mcp.json
└── .github/
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

---

## Agent and Skills

### Main Agent

`.github/agents/main.agent.md` defines the behavior and orchestration for:

- Diagnosing migration failures on backlog issues
- Planning wave-based migrations
- Validating `config/warp.yml` updates

### diagnose-migration

Folder: `.github/skills/diagnose-migration`
- Main instructions: `SKILL.md`
- Classification reference: `failure-catalog.md`
- Purpose: classify failed/stuck migration issues from issue body, labels, and comments
- Output behavior: post a diagnosis comment to the issue with root cause, remediation, and retry command (typically `/migrate`)

### plan-migration

Folder: `.github/skills/plan-migration`

- Main instructions: `SKILL.md`
- Scoring rules: `scoring.md`
- Document format: `output-template.md`
- Purpose: score repositories, assign migration waves, apply `planning:*` labels, and generate a customer-ready migration strategy document
- Output behavior: generates `migration-strategy.md` (default location: repository root unless the user specifies otherwise)

### validate-config

Folder: `.github/skills/validate-config`

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
# Install the plugin locally
copilot plugin install ./hello-warp-plugin

# Verify
copilot plugin list

# Interactive session
copilot
/agent              # confirm warp-migration-agent loaded
/skills list        # confirm skills loaded
```

### Deploy as an Agentic App

1. **Push to a public GitHub repository:**
   ```bash
   cd hello-warp-plugin
   git init && git add . && git commit -m "Warp migration agent plugin"
   gh repo create packfiles/warp-migration-agent --public --source=. --push
   ```

2. **Register a GitHub App** at GitHub Settings → Developer settings → GitHub Apps:
   - Permissions needed: Issues (read/write), Pull Requests (read/write), Contents (read)
   - Subscribe to events: `issues`, `issue_comment`, `pull_request`, `pull_request_review_comment`

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

## License

MIT
