---
name: diagnose-migration
description: Diagnoses a failed or stuck migration by reading the current backlog issue's body, labels,
  and comments, classifying the root cause, and recommending specific remediation steps.
  Invoked on a Migration HQ backlog issue where migration has failed or is not progressing.
---

## Your Approach

- **Never fabricate error causes.** Classify failures only from evidence present in the
  issue body, labels, and comments. If the evidence is insufficient, say so explicitly and
  ask for what is missing.
- **Never fabricate product information.** All claims about Warp behavior, limits, and
  remediation steps must come from `https://kb.packfiles.io`.
- Be concise and actionable. The engineer reading this diagnosis needs to know exactly what
  went wrong and exactly what to do next — not a wall of possibilities.

---

When invoked on a migration issue that has failed:

## Step 1: Read the Issue in Full

Use GitHub MCP tools to gather all context from the current backlog issue.

### 1a — Issue body and labels

Use `issue_read` to retrieve:
- The full issue body (contains repository metadata: size, activity, PR count, pipeline
  count, last activity date, and any outstanding tasks)
- All labels, which indicate:
  - Provider: `azure-devops` or `bitbucket-server`
  - Size flag: `too-big` (repository exceeds 10 GB)
  - Activity level: `active`, `low-activity`, `inactive`
  - ADO pipeline presence: `azure-pipelines`
  - Any status labels (e.g., `status:failed`, `status:in-progress`)

### 1b — Comments

Use `get_comments` to retrieve all comments on the issue, sorted oldest-first.
Scan every comment for:
- Runner log output or error messages posted by Warp after a migration attempt (look for
  a **Troubleshoot** section containing a **Warp runner logs** link)
- Any prior diagnosis or manual notes left by engineers
- Slash commands that were previously invoked (e.g., `/migrate`)

### 1c — Determine provider

- Label `azure-devops` → **Azure DevOps (ADO)**
- Label `bitbucket-server` → **Bitbucket Server (BBS)**

If neither label is present, this issue is not a standard backlog issue. Inform the user
and halt — do not attempt diagnosis.

### 1d — Check if a migration was actually attempted

If no prior `/migrate` comment is found and no error output is present, inform the user:

> "No migration attempt appears to have been made on this issue yet. Run `/migrate` as a
> comment on this issue to start the migration."

---

## Step 2: Classify the Failure

Read `skills/diagnose-migration/failure-catalog.md` in full before attempting to
classify the failure. Apply the classification criteria exactly as written there.

Match the error output and context gathered in Step 1 against the failure catalog. A single
migration failure may match more than one category — list all that apply, ordered from most
to least likely based on the available evidence.

**Credential failures have three distinct sub-categories** — distinguish between them:
- `credentials-insufficient-permissions` — wrong scopes on a valid, non-expired PAT
- `credentials-expired-pat` — PAT has passed its expiration date
- `credentials-outdated-vault` — vault file in Migration HQ does not reflect current
  credentials

If the failure pattern is not covered by the catalog, query the KB dynamically before
concluding the failure is unclassified:

```
GET https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration.md?ask=<question>
```

Use a specific, self-contained natural-language question as the `ask` parameter. 

If the log output is truncated, incomplete, or absent, do not guess. Move to Step 4
(ask for more context) instead.

Record for each matched category:
- Category identifier (e.g., `credentials-expired-pat`, `size-limit`)
- The specific evidence from the issue that supports this classification
- Confidence: High / Medium / Low

---

## Step 3: Recommend Resolution

For each classified failure category, produce:

1. **Root cause** — one sentence stating exactly what went wrong
2. **Remediation steps** — a numbered list of specific, actionable steps
3. **Retry instruction** — the exact slash command to post as an issue comment to retry
   the migration once remediation is complete. The retry command is always `/migrate`
   unless the failure catalog specifies otherwise.
If multiple categories matched, present them in order of confidence. Clearly distinguish
between "fix this first" (blocking) and "also review" (non-blocking but relevant).

**Special rule — `main-branch` category:** If the `main-branch` category is classified,
present it as the sole blocking issue. There is no point diagnosing per-repository
failures until the Migration HQ default branch is corrected.

Do not recommend remediation steps you cannot source from `https://kb.packfiles.io` or
from the evidence in the issue itself.

---

## Step 4: Handle Ambiguity

If the failure cannot be classified with at least Medium confidence from available
evidence, do not produce a speculative diagnosis. Instead, ask the user for the specific
information needed to make a determination:

### If runner logs are absent or truncated
 
Direct the user to locate the full logs using the following path:
 
1. If the issue's failure comment includes a **Troubleshoot** section, expand it and click
   the **Warp runner logs** link — this goes directly to the failed runner run
2. Alternatively, open the **Actions** tab of Migration HQ and click on the failed run
3. On the run's details page, click the **Packfiles Warp Runner ...** button to view the
   full log collection
4. Use the search field (upper right) and search for `Error` to locate the relevant failure
   message
5. Share the full error output so diagnosis can proceed

Ask only for what is actually needed — do not request information that is already
available in the issue.

---

## Step 5: Post the Diagnosis

Use `add_issue_comment` to post the diagnosis directly on the issue as a comment. Do
not only return the diagnosis as a chat response — it must be posted to the issue so it
is visible to the whole team and part of the issue's permanent record.

Format the comment as follows. Adapt the structure based on how many categories were
identified:

```markdown
## Migration Diagnosis

**Repository:** <repo name> (<provider>)
**Failure classified as:** <category name(s)>

---

### Root Cause

<one sentence>

### Remediation Steps

1. <step>
2. <step>
...

### Retry

Once remediation is complete, post the following comment on this issue:

```
/migrate
```

---
*Diagnosed by the Warp Migration Assistant. If this diagnosis does not match what you
are seeing, share the full runner logs from the Actions tab and re-invoke the agent.*
```

If the failure is ambiguous (Step 4), post the clarifying question as the comment instead
of a diagnosis. Do not post a partial diagnosis alongside an ambiguity question.

---

## Output Requirements

- Always post the diagnosis to the issue via `add_issue_comment` — do not only return
  it in the chat conversation, and do not shell out to `gh` or
  `curl` (those calls hit a firewall block in Cloud Agent's runtime).
- Keep root cause statements to one sentence. Do not pad with hedges or qualifications.
- Remediation steps must be specific: reference exact Warp Vault settings, exact slash
  commands, and exact KB URLs where relevant.
- If the issue has the `too-big` label, always address size in the diagnosis even if
  another failure category also matched.
- If the `main-branch` category is identified, do not diagnose per-repository failures
  alongside it — fix the environment first.
- Do not suggest contacting Packfiles support as a first step. Only suggest it if the
  failure cannot be classified and the user has already provided complete runner logs.
  Direct users to Product Support via the Support tab in their Warp Project.