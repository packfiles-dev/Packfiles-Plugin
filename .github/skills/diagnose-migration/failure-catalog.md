# Failure Catalog for diagnose-migration

Loaded by `.github/skills/diagnose-migration/SKILL.md` during Step 2. Match failure evidence against
the categories below. Apply only classifications that are supported by actual evidence
from the issue — do not speculatively assign categories.

All remediation guidance in this catalog is sourced from the official Packfiles KB at
https://kb.packfiles.io. If a failure pattern is not covered here, do not invent
remediation steps — follow Step 4 of the skill instead.

---

## Category: `credentials-insufficient-permissions`

**Symptoms in logs:**
- "Insufficient permissions", "403 Forbidden", "Access denied", "not authorized"
- Error messages referencing authentication or authorization in runner logs

**Provider:** ADO and BBS (source credentials); also applies to destination GitHub PAT

**Root cause:** The personal access token (PAT) stored in Warp Vault does not have the
required scopes to read from the source platform or write to the destination GitHub
organization, or the Warp GitHub App installation is missing permissions on the destination
organization.

**Remediation:**
1. Verify the PAT has the required scopes for your source platform. Refer to the
   [Set Up Your Vault](https://kb.packfiles.io/guides/quickstart/set-up-your-vault) guide
   for the specific scopes needed for your provider.
2. For destination failures, verify the Warp GitHub App has the necessary permissions on
   the destination organization.
3. If scopes are incorrect, generate a new PAT with the correct permissions.
4. Open the Warp Vault desktop application and update the credential with the new PAT.
5. Push the updated `vault.age` file to Migration HQ.
6. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration

---

## Category: `credentials-expired-pat`

**Symptoms in logs:**
- "Invalid credentials", "Authentication failed", "401 Unauthorized", "Token has expired"
- "PAT is invalid", "Could not authenticate to <source system>"
- Authentication errors that appear despite the vault appearing correctly configured

**Provider:** ADO and BBS (source credentials); also applies to destination GitHub PAT

**Root cause:** The personal access token (PAT) for the source or destination platform has
passed its expiration date and is no longer valid.

**Remediation:**
1. Check the expiration date of the PAT on your source platform (ADO or BBS) and on GitHub.
2. Generate a new PAT with the same scopes on the affected platform(s).
3. Open the Warp Vault desktop application and update the credential with the new PAT.
4. Push the updated `vault.age` file to Migration HQ.
5. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration

---

## Category: `credentials-outdated-vault`

**Symptoms in logs / issue:**
- Authentication errors that appear despite credentials being recently rotated
- Credentials were updated outside of Warp Vault, or a new vault was generated but not
  pushed to Migration HQ

**Provider:** ADO and BBS

**Root cause:** The `vault.age` file in Migration HQ is stale — credentials were updated
outside of Warp Vault or the vault was regenerated but not pushed.

**Remediation:**
1. Open the Warp Vault desktop application and verify your credentials are current.
2. Re-export the vault file.
3. Push the updated `vault.age` file to Migration HQ.
4. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration

---

## Category: `size-limit`

**Symptoms in logs or issue:**
- "Repository size exceeds", "exceeds the maximum allowed size", "40 GB limit"
- `too-big` label is present on the issue
- Issue body shows repository size ≥ 40 GB

**Provider:** ADO and BBS

**Root cause:** The repository's size meets or exceeds Warp's 40 GB absolute size limit
and cannot be automatically migrated without remediation.

**Note on `too-big` label:** Warp automatically assigns the `too-big` label to any
repository that exceeds size limits. Repositories ≥ 40 GB are hard-blocked. Repositories
between 10 GB and 40 GB may be flagged but can still be attempted — review individually.

**Remediation:**
1. Review the size limitations for your source platform:
   - ADO: https://kb.packfiles.io/migrations/azure-devops/limitations
   - BBS: https://kb.packfiles.io/migrations/bitbucket-server/limitations
2. Assess what is driving the repository size (large binaries, bloated history, LFS objects).
3. If the history contains committed large files, rewrite history to remove in the source system before migrating.
4. If LFS is in use, see the `lfs` category below — LFS objects are not migrated by Warp.
5. After remediation, confirm the repository size is below 40 GB.
6. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `file-size`

**Symptoms in logs:**
- "File exceeds maximum size", "single file too large", "400 MB limit", "400mb"
- References to a specific file path that is oversized

**Provider:** ADO and BBS

**Root cause:** One or more individual files in the repository exceed Warp's 400 MB
single-file size limit.

**Remediation:**
1. Identify the file(s) named in the error output.
2. Remove the file from the repository history in the source system.
3. If the file is still needed, migrate it separately outside of the repository (e.g.,
   as a release artifact or via Git LFS after migration is complete).
4. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `commit-size`

**Symptoms in logs:**
- "Commit too large", "commit size exceeds", "2 GB", "push rejected"

**Provider:** ADO and BBS

**Root cause:** One or more commits in the repository's history exceed Warp's 2 GB
per-commit size limit.

**Remediation:**
1. Identify the commit(s) named in the error output using `git show --stat <commit-sha>`.
2. Rewrite the affected commits in the source repository to remove large objects.
3. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `git-ref`

**Symptoms in logs:**
- "Reference name too long", "ref exceeds maximum length", "255 bytes", "invalid ref"
- References to specific branch or tag names

**Provider:** ADO and BBS

**Root cause:** One or more branch or tag names in the repository exceed Warp's 255-byte
Git reference length limit.

**Remediation:**
1. Identify the branch or tag names mentioned in the error.
2. Rename or delete the affected branches or tags in the source system.
3. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `lfs`

**Symptoms in logs or issue:**
- "LFS", "Git LFS", "large file storage", "lfs pointer", "smudge filter"
- Repository size is anomalously large relative to commit count
- `too-big` label present on a repository with relatively few commits

**Provider:** ADO and BBS

**Root cause:** The repository uses Git LFS for large file storage. Warp does not currently
support migrating LFS objects — only the LFS pointer files are migrated, not the objects
they reference.

**Remediation:**
1. Confirm LFS usage by running `git lfs ls-files` in a local clone of the source
   repository.
2. Decide on a strategy:
   - **Accept pointer-only migration:** Migrate the repository knowing LFS objects will
     not transfer. Store LFS objects separately and reconfigure LFS after migration.
   - **Convert LFS to regular files:** Use `git lfs migrate export` to rewrite history
     with LFS objects embedded as regular files (this will likely increase repository
     size significantly — confirm the result is still under 40 GB).
3. Proceed accordingly, then retry with `/migrate`.
4. Monitor Warp's roadmap for LFS support: https://kb.packfiles.io

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `metadata-size`

**Symptoms in logs:**
- "Metadata size exceeds", "issue metadata too large", "PR metadata", "20 GB metadata limit"

**Provider:** ADO and BBS

**Root cause:** The repository's associated metadata — issues, pull requests, releases,
and attachments — exceeds Warp's 20 GB metadata size limit.

**Remediation:**
1. Identify which metadata type is driving the size (typically PR or release attachments).
2. Archive or remove large attachments from pull requests or releases in the source system
   before migrating.
3. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

---

## Category: `invalid-destination-name`

**Symptoms in logs:**
- "Name cannot end in .wiki", "invalid repository name", "repository name is not permitted"
- "422", "Could not create repository"
- Issue body shows a destination name ending in `.wiki` or containing other invalid characters

**Provider:** ADO and BBS

**Root cause:** The intended destination repository name violates a GitHub naming
constraint. A common example is a name ending in `.wiki` — GitHub automatically creates
a wiki for each repository using `<repo-name>.wiki`, so no repository may be named with
that suffix.

**Remediation:**
1. Post `/rename-destination <new-name>` as a comment on this issue to assign a valid
   destination name.
2. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration
- https://kb.packfiles.io/using-warp/migration-hq/runner-agent

---

## Category: `conflict`

**Symptoms in logs:**
- "Repository already exists", "destination already exists", "name conflict"
- "Could not create repository", "repository name is taken"

**Provider:** ADO and BBS

**Root cause:** A repository with the intended destination name already exists in the
target GitHub organization (distinct from an invalid name — the name is valid but taken).

**Remediation — option A (rename destination):**
1. Post `/rename-destination <new-name>` as a comment on this issue.
2. Retry with `/migrate`.

**Remediation — option B (remove conflicting repo):**
1. If the existing GitHub repository is a previous failed migration attempt with no data
   you need to keep, delete it from GitHub.
2. Retry with `/migrate`.

**Source:** https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration

---

## Category: `main-branch`

**Symptoms in issue / Actions tab:**
- Runner Agent fails immediately after any Warp operation.
- No workflow runs appear in Migration HQ's Actions tab, or they fail during setup.
- Warp does not respond to slash commands at all.
- This pattern affects **all** issues in the backlog, not just one repository.

**Provider:** ADO and BBS (environment-level issue, not repository-specific)

**Root cause:** Warp requires the Migration HQ repository's default branch to be named
`main`. If the GitHub organization enforces a different default branch name (e.g.,
`master`, `develop`, `trunk`), the Warp Runner Agent will fail to execute any workflow.

**Remediation:**
1. Navigate to the Migration HQ repository on GitHub.
2. Go to **Settings** > **General**.
3. Under **Default branch**, change the branch name to `main`.

> **Note:** If your organization enforces a branch naming policy, you can temporarily
> change the organization-level default to `main`, allow Migration HQ to operate, and
> then revert the policy. Migration HQ will retain `main` as its default branch.

**Source:** https://kb.packfiles.io/using-warp/support/troubleshooting/main-branch-requirement

---

## Category: `github-outage`

**Symptoms in issue / Actions tab:**
- Migration fails with no repository-specific error.
- Runner Agent cannot execute workflows at all.
- Failures are occurring broadly across multiple issues simultaneously.

**Provider:** ADO and BBS (external dependency, not provider-specific)

**Root cause:** GitHub is experiencing an outage that prevents Runner Agents from
executing workflows.

**Remediation:**
1. Check [GitHub Status](https://www.githubstatus.com/) for any ongoing incidents.
2. Wait for the incident to be resolved.
3. Retry with `/migrate`.

**Source:**
- https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration

---

## Unclassified Failures

If the log output does not match any of the above categories, do not speculate. Follow
Step 4 of the skill (ask for more context) and direct the user to:
- https://kb.packfiles.io for product documentation
- The **Actions** tab of Migration HQ for full runner logs (see
  https://kb.packfiles.io/using-warp/migration-hq/runner-agent for how to read them)
- Product Support via the Support tab in their Warp Project, if full logs are available
  and the failure still cannot be classified

> **Dynamic KB queries:** If you need additional information not covered in this catalog,
> query the KB dynamically:
> `GET https://kb.packfiles.io/using-warp/support/troubleshooting/failed-migration.md?ask=<question>`
> Use a specific, self-contained natural-language question as the `ask` parameter.