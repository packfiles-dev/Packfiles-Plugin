# Packfiles Migration Agent Plugin

An Agentic App plugin for Packfiles Warp migration workflows. It helps teams plan migrations, diagnose failed/stuck migrations, and validate Migration HQ configuration for Azure DevOps and Bitbucket Server to GitHub migrations.

## Plugin Structure

```
.
├── plugin.json
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

## Security

This repository is public because the contents are synced and run as a deployed
Agentic App. Every file here defines production agent behavior, and the agent
processes untrusted issue/PR/comment content. The plugin stores **no secrets** —
it authenticates to the Warp API via OIDC. See [`SECURITY.md`](SECURITY.md) for
the security model and how to report a vulnerability, and
[`.github/CODEOWNERS`](.github/CODEOWNERS) for required review on agent changes.
