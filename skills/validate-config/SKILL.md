---
name: validate-config
description: Reads the current config/warp.yml from Migration HQ, validates it against the documented
  Warp configuration schema, detects conflicts and security concerns, and posts a structured
  findings report as a PR comment. Can also help draft or fix specific configuration sections on request.
---

## Your Approach

- **Never invent schema rules.** All validation logic must come from
  `.github/skills/validate-config/warp-yml-schema.md` or from `https://kb.packfiles.io`.
  Do not apply constraints that are not documented.
- **Be precise about severity.** An error means the configuration will not work as intended
  or will cause Warp to malfunction. A warning means the configuration is valid but has
  implications the user should be aware of. Not every imperfection is an error.
- **Never fabricate product information.** If you are unsure whether a configuration value
  or behavior is correct, query `https://kb.packfiles.io/using-warp/migration-hq/warp.yml.md`
  before making a claim.

---

## Step 1: Determine the Invocation Context

The user may invoke this skill in one of two ways:

**A — Review mode (no specific request):** The user asks to "check", "review", or
"validate" the config file. Proceed through all steps.

**B — Authoring mode (specific request):** The user asks to "add", "write", "update",
or "fix" a specific part of the config. Read the current file (Step 2), then jump
directly to producing the requested change (Step 3), skipping full validation unless
the user asks for it.

If the request is ambiguous, default to Review mode.

---

## Step 2: Read the Current Configuration

Read `config/warp.yml` from the current repository (Migration HQ).

If the file does not exist:
> "No `config/warp.yml` file was found in this repository. Warp creates a default version
> when a project is initialized. If you would like, I can help you create one from scratch
> — just tell me what you need to configure."

If the file exists but is entirely commented out (only the default template with no
uncommented configuration keys besides `version`), acknowledge this:
> "The `config/warp.yml` file contains only the default template with no active
> configuration. This is valid — Warp uses its defaults for all settings. If you would
> like to configure something specific, I can help."

Otherwise, parse the YAML content and proceed.

---

## Step 3: Validate Against Schema

Read `skills/validate-config/warp-yml-schema.md` in full before performing
any validation. Apply the rules exactly as written there.

For each top-level configuration key present in the file, validate:
- All required fields are present
- All values are within the documented set of valid options
- No mutually exclusive options are combined
- Referenced resources exist (e.g., script filenames in `bin/`, team slugs that appear
  to follow the expected format)

Record every finding as one of:
- **✅ Valid** — correct and no concerns
- **⚠️ Warning** — valid syntax but has implications worth noting
- **❌ Error** — incorrect, invalid value, or will prevent Warp from functioning correctly

---

## Step 4: Check for Security Concerns

After schema validation, review the configuration for the following security implications.
These are always surfaced as warnings (not errors) since they may be intentional.

- **`target_repo_visibility: "internal"`** — migrated repositories will be visible to all
  members of the GitHub organization. Confirm this is intentional.
- **No policies defined** — all organization members can run all Warp commands. If the
  migration involves sensitive repositories or phased access, consider adding policies.
- **`allow_vault: true` on a script** — that script has access to all credentials stored
  in Warp Vault. Confirm the script genuinely requires vault access.
- **Policies defined for some commands but not others** — defining any slash command policy
  causes all unlisted commands to be denied. Confirm the unlisted commands are intentionally
  disabled.
- **Global issue policy (no `labels`)** — grants broad access across all issues. Confirm
  this is the intended scope.

---

## Step 5: Post Findings

**Review mode:** Use engine-tools-reply_to_comment to post the full findings report on the current
PR. Format:

```markdown
## Warp Configuration Review

**File:** `config/warp.yml`
**Reviewed at:** <timestamp>

---

### Summary

| Severity | Count |
|---|---|
| ✅ Valid | N |
| ⚠️ Warning | N |
| ❌ Error | N |

---

### Findings

#### <Top-Level Key>
- ✅ / ⚠️ / ❌ **<field or rule>** — <explanation>

<repeat for each key and finding>

---

### Recommendations

<numbered list of the most impactful changes, if any — omit section entirely if no
errors or warnings were found>

---
*Reviewed by the Warp Migration Assistant. Reference: https://kb.packfiles.io/using-warp/migration-hq/warp.yml*
```

If there are no errors and no warnings, say so clearly:
> "No issues found. Your `config/warp.yml` configuration is valid."

**Authoring mode:** Return the requested configuration snippet directly in chat. Do not
post it as an issue comment unless the user asks. Use the schema sub-file to ensure the
snippet is correct before returning it. Accompany the snippet with a brief explanation of
what each relevant field does.

---

## Step 6: Offer to Apply Fixes (Review Mode Only)

After posting findings, if errors or warnings were found, offer:
> "Would you like me to suggest corrected YAML for any of these issues, or apply any of
> the recommended changes directly?"

If the user confirms, produce the corrected YAML snippet for each error. Do not use
`update_file` to directly overwrite `config/warp.yml` without the user's explicit
confirmation — always show the proposed change first.

---

## Output Requirements

- In Review mode, always post findings as an issue comment via engine-tools-reply_to_comment.
  Do not only return findings in the chat conversation.
- In Authoring mode, return the snippet in chat. Post to the issue only if asked.
- Keep explanations for each finding to one or two sentences — sufficient to understand
  the problem and the fix without over-explanation.
- Do not flag the default `version: "1.0"` as a finding — it is always expected to be
  present.
- If the file is empty or only whitespace, treat it as an error (Warp requires at least
  the `version` key).
