# Security Policy

## Reporting a Vulnerability

If you discover a security issue in this plugin, please report it **privately**:

- Use GitHub's **private vulnerability reporting** — Security tab → *Report a
  vulnerability*, or
- Email **security@packfiles.io** (or **support@packfiles.io**).

Please do **not** open a public issue for security reports. We aim to
acknowledge reports within 3 business days.

## Security Model

This repository is a public GitHub Agent HQ **plugin**. Its contents are synced
and executed as the instructions of a deployed Agentic App. Treat every change
to `agents/`, `skills/`, and `plugin.json` as a
change to **production agent behavior**.

- **No secrets belong in this repo.** The plugin authenticates to the Warp API
  via OIDC workload-identity federation — no tokens, PATs, or long-lived
  credentials are ever stored here.
- **Untrusted input.** The agent processes issue, PR, and comment content,
  labels, runner logs, and source-repository metadata, all of which are
  untrusted. The agent prompt instructs the model to treat that content as data,
  never as instructions (see `agents/main.agent.md`).
- **Branch integrity.** The default branch (`main`) is what the platform syncs.
  It is protected, and changes to agent, skill, and MCP/plugin files require
  maintainer review (see `.github/CODEOWNERS`).
