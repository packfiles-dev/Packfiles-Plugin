# warp.yml Schema Reference for validate-config

Loaded by `skills/validate-config/SKILL.md` during Step 3. Apply these rules exactly as written.
All constraints are sourced from https://kb.packfiles.io/using-warp/migration-hq/warp.yml
and its linked pages.

---

## Top-Level Structure

```yaml
version: "1.0"                     # required
target_repo_visibility: "private"  # optional
azure_devops:                      # optional; ADO-specific
  global:
    azure_pipelines_github_app_service_connection_id: <string>
  <org-slug>:
    azure_pipelines_github_app_service_connection_id: <string>
policies:                          # optional
  issues:
    <policy-id>:
      labels: [...]
      teams: [...]
      users: [...]
  slash_commands:
    /<command>:
      teams: [...]
      users: [...]
  scripts:
    <script-id>:
      teams: [...]
      users: [...]
custom_tasks:                      # optional
  <task-id>:
    title: <string>
    body: <string>
    labels: [...]
scripts:                           # optional
  <script-id>:
    name: <string>
    description: <string>
    language: <bash|pwsh|python|ruby>
    filename: <string>
    allow_vault: <boolean>
    arguments:
      <arg-name>:
        description: <string>
        required: <boolean>
```

---

## `version`

- **Required.** Must be present and set to `"1.0"`.
- **Error** if missing or set to any other value.

---

## `target_repo_visibility`

- **Optional.** Controls visibility of repositories created by Warp during migration.
- **Valid values:** `"private"` (default), `"internal"`
- `"private"` — repositories accessible only to users with explicit access
- `"internal"` — repositories visible to all members of the GitHub organization
- **Requires GitHub Enterprise Cloud or GitHub Enterprise Server** for `"internal"`
- **Does not affect** repositories already migrated — only new repositories created by Warp
- **Error** if set to any value other than `"private"` or `"internal"`
- **Warning** if set to `"internal"` — confirm this is intentional (broad visibility)

---

## `azure_devops`

- **Optional.** Only relevant when migrating from Azure DevOps and using `/rewire-pipeline`
  or `/rewire-all-pipelines`.
- **Structure:** `global` key provides a default; named org-slug keys provide per-org overrides.
- **`azure_pipelines_github_app_service_connection_id`** — string; the service connection ID
  from Azure DevOps (found under Project Settings → Service connections)
- **Warning** if `azure_devops` is present but no `azure_pipelines_github_app_service_connection_id`
  is set under `global` or any org key — the block has no effect without a value
- **Warning** if a named key under `azure_devops` does not appear to be a valid org slug
  (contains spaces, uppercase characters, or special characters other than `-`)

---

## `policies`

Source: https://kb.packfiles.io/using-warp/migration-hq/access-policies

### Activation behavior

- **No policies defined:** all organization members can run all commands — this is the
  default open state.
- **Any issue policy defined:** Warp enters restrictive mode for issues. Issues not matched
  by any policy are inaccessible.
- **Any slash command policy defined:** all slash commands not explicitly listed are denied.
  **Error** if slash command policies are defined but `/migrate` is not listed — users will
  be unable to start migrations.

### `policies.issues.<policy-id>`

- **`labels`** (optional) — array of label strings. If omitted, policy applies to all
  issues globally.
- **`teams`** (optional) — array of GitHub team slugs. Must be lowercase and match the
  exact GitHub team slug.
- **`users`** (optional) — array of GitHub usernames.
- **Error** if a policy entry has neither `teams` nor `users` — the policy grants access
  to nobody and is a no-op.
- **Warning** if policy identifiers are not unique (YAML will silently overwrite duplicates).
- **Warning** if `labels` references a label that is not a known Warp-generated label and
  does not appear to be a planning wave label — may be a typo.

### `policies.slash_commands.<command>`

- The command key must start with `/`.
- **Error** if a command key does not start with `/`.
- **Error** if a policy entry has neither `teams` nor `users`.
- **Warning** if slash command policies are defined and `/help` is not listed — users will
  be unable to use `/help`.
- **Warning** if slash command policies are defined and `/refresh` is not listed.
- Known valid commands (from KB): `/migrate`, `/rename-destination`, `/refresh`, `/run`,
  `/help`, `/add-team`, `/rewire-pipeline`, `/rewire-all-pipelines`, `/integrate-boards`,
  `/autolink-work-items`, `/lock-ado-repo`, `/disable-ado-repo`
- **Warning** if a command key is not in the known valid command list — may be a typo.

### `policies.scripts.<script-id>`

- The script ID must match a key defined under the top-level `scripts` block.
- **Warning** if a script policy references a script ID not defined in `scripts`.

---

## `custom_tasks`

Source: https://kb.packfiles.io/using-warp/migration-hq/custom-tasks

### `custom_tasks.<task-id>`

- **`title`** — required string. The heading shown in the issue body.
- **`body`** — required string. Supports full Markdown including checklists.
- **`labels`** — optional array of label strings. If omitted, the task appears on every
  issue in the project (global task).
- **Error** if `title` or `body` is missing.
- **Warning** if `labels` is an empty array `[]` — this is neither a global task (no
  `labels` key) nor a label-specific task. Behavior is undefined; omit the key entirely
  for global tasks.
- **Warning** if task identifiers are not unique (YAML will silently overwrite duplicates).
- **Warning** if `labels` references a label not recognized as a Warp-generated or
  planning-wave label — may be a typo that causes the task to never trigger.
- **Note:** Warp automatically overwrites the issue body when labels change. Remind the
  user that manual edits to issue bodies may be overwritten.

---

## `scripts`

Source: https://kb.packfiles.io/using-warp/migration-hq/custom-scripts

### `scripts.<script-id>`

**Required fields:**
- **`name`** — string; human-readable display name
- **`description`** — string; brief explanation
- **`language`** — must be one of: `bash`, `pwsh`, `python`, `ruby`
- **`filename`** — string; name of the script file in the `bin/` directory of Migration HQ

**Optional fields:**
- **`allow_vault`** — boolean (default: `false`); injects `PKFS_MASTER_KEY` into the
  script environment, giving access to all Warp Vault credentials
- **`arguments`** — map of named arguments

**Validation rules:**
- **Error** if any required field is missing.
- **Error** if `language` is not one of the four valid values.
- **Warning** if `allow_vault: true` — confirm the script genuinely requires vault access,
  as this exposes all credentials.
- **Warning** if the `filename` value does not have the extension expected for the declared
  `language` (e.g., `.sh` for `bash`, `.py` for `python`, `.rb` for `ruby`, `.ps1` for `pwsh`)
- **Warning** if the script file named in `filename` cannot be confirmed to exist in
  `bin/` (the skill cannot always verify this — note the limitation if uncertain).

### `scripts.<script-id>.arguments.<arg-name>`

- **`description`** — required string.
- **`required`** — optional boolean (default: `false`).
- **Error** if `description` is missing.
- Argument names are passed as environment variables prefixed `WARP_SCRIPT_ARGS__` with
  the name uppercased and non-alphanumeric characters replaced by underscores.

---

## Complete Valid Example

For reference when generating corrected YAML:

```yaml
version: "1.0"

target_repo_visibility: "internal"

azure_devops:
  global:
    azure_pipelines_github_app_service_connection_id: "abc12345-..."
  my-ado-org:
    azure_pipelines_github_app_service_connection_id: "def67890-..."

policies:
  issues:
    global_access:
      teams: ["release-engineering"]
    wave_1_access:
      labels: ["planning:wave-1"]
      teams: ["wave-1-team"]
  slash_commands:
    /help:
      teams: ["all-contributors"]
    /migrate:
      teams: ["release-engineering", "wave-1-team"]
    /refresh:
      teams: ["all-contributors"]
    /rewire-pipeline:
      teams: ["release-engineering"]
    /lock-ado-repo:
      teams: ["release-engineering"]
  scripts:
    notify_team:
      teams: ["release-engineering"]

custom_tasks:
  wave_1_checklist:
    title: "Wave 1 Migration Checklist"
    body: |
      - [ ] Run /migrate
      - [ ] Verify migrated repository
      - [ ] Run /lock-ado-repo
    labels: ["planning:wave-1"]

scripts:
  notify_team:
    name: "Notify Team"
    description: "Sends a Slack notification when a migration completes"
    language: bash
    filename: "notify_team.sh"
    allow_vault: false
```