# Output Template for plan-migration

Loaded by `skills/plan-migration/SKILL.md` during Step 7. Produce the migration strategy document
using exactly the section structure below. All 11 sections are required. Do not add, remove,
rename, or merge sections. Omit only provider-specific *subsections* that do not apply to
the current inventory — do not omit entire numbered sections.

---

```
# Migration Strategy: [Provider(s)] to GitHub
### Powered by Packfiles Warp  |  [Current Date]
```

---

### SECTION 1 — Executive Summary

A professional 4–6 sentence narrative summarizing: migration scope, wave distribution, estimated total duration, and the use of Packfiles Warp. Follow with:

**Migration Summary**

| Metric | Value |
|---|---|
| Total Repositories | |
| Azure DevOps Repositories | N *(omit row if no ADO repos)* |
| Bitbucket Server Repositories | N *(omit row if no BBS repos)* |
| Wave 1 — Low Risk | N (X%) |
| Wave 2 — Medium Risk | N (X%) |
| Wave 3 — High Risk | N (X%) |
| Manual Review Required | N (X%) |
| Repositories Exceeding 40 GB Warp Limit | |
| Estimated Total Migration Duration | |
| Migration Tool | Packfiles Warp |

---

### SECTION 2 — Repository Inventory Overview

A narrative paragraph describing the overall inventory composition. Then:

**Inventory by [Team (if ADO), BBS (if BBS), Team/BBS (if both ADO/BBS present)] Project**

If both providers are present, include a Source column. If only one provider is present,
omit the Source column.

If only ADO is present, the Organization column should be present. If only BBS is present, the Organization column should be omitted. If both are present, the Organization column should be present, but BBS values should be blank.

If only ADO is present, the Team Project column header should be "Team Project". If only BBS is present, the column header should be "BBS Project". If both are present, the column header should be "Team Project / BBS Project".

| Source | Organization | Team Project / BBS Project | Repository Count | Total Size (GB) | Active Repos | Avg PRs/Repo |
|---|---|---|---|---|---|---|

Include a Totals row.

**Size Distribution**

| Size Band | Repository Count | Percentage |
|---|---|---|
| Excellent (< 10 MB) | | |
| Very Good (10 MB – < 100 MB) | | |
| Good (100 MB – < 500 MB) | | |
| Fair (500 MB – < 5 GB) | | |
| Poor (5 GB – < 40 GB) | | |
| Exceeds Warp Limit (≥ 40 GB) | | |
| **Total** | | 100% |

---

### SECTION 3 — Repository Scoring and Wave Assignment

A paragraph explaining the three-component scoring methodology, what each component measures, and how wave assignments are made. Explicitly note that the Simplicity Score differs by provider IF both providers are present: BBS repositories are scored on PR count only (no pipelines), while ADO repositories are scored on PR count and pipeline count.

**Wave Summary**

| Wave | Score Range | Risk Level | Count | % of Total | Characteristics |
|---|---|---|---|---|---|
| Wave 1 | 9–11 | Low Risk | | | Small, inactive or low-activity, simple |
| Wave 2 | 6–8 | Medium Risk | | | Standard size and complexity |
| Wave 3 | 3–5 | High Risk | | | Larger or more complex repositories |
| Manual Review | 0–2 or override | Very High Risk | | | Requires individual pre-migration assessment |
| **Total** | | | | 100% | |

If both providers are present, add:

**Wave Distribution by Provider**

| Wave | Azure DevOps | Bitbucket Server | Total |
|---|---|---|---|

**Score Component Reference**

Present the scoring table (two tables if both providers are present, with a title indicating the provider) with the headers below to show how each component is measured.

| Component | Criteria | Points |

---

### SECTION 4 — Migration Wave Strategy

For each of the four waves produce a subsection (`#### Wave 1`, `#### Wave 2`,
`#### Wave 3`, `#### Manual Review`) containing:

1. A 2–3 sentence description of the wave's characteristics
2. A Wave Details table:

| Attribute | Value |
|---|---|
| Repository Count | |
| Estimated Duration | |
| Daily Throughput | |
| Success Criteria | |

3. A bulleted list of special considerations or actionable recommendations specific to that wave, derived from the inventory data. For example:
   - For waves containing ADO repos: note pipeline-related post-migration work where count > 0
   - For waves containing BBS repos: note that only `/migrate` and universal commands apply
     (ADO pipeline-rewiring commands do not apply to BBS)
   - For Manual Review: include the remediation-and-re-score path
   - Team communication recommendations

After all four subsections:

**Total Migration Timeline**

| Wave | Count | % of Total | Business Days | Cumulative Business Days |
|---|---|---|---|---|
| Wave 1 | | | | |
| Wave 2 | | | | |
| Wave 3 | | | | |
| Manual Review | | | | |
| **Total** | | 100% | | |

Business days are calculated as `ceil(wave_count / daily_throughput)`.

**Wave Labels in Migration HQ**

A paragraph explaining that planning wave labels are applied to every open backlog issue
automatically by this agent. These labels allow engineers to filter issues by wave using
GitHub's native label search.

---

### SECTION 5 — Repository Size Analysis

A brief narrative on size-related risks specific to this inventory. Then:

**Top 10 Largest Repositories**

Include Source column only if both providers are present.

| # | Source | Organization / Project / Repository | Size (GB) | Last Activity | Commits/Year | PR Count | Wave |
|---|---|---|---|---|---|---|---|

**Size Risk Assessment**

- Repositories ≥ 40 GB: list each by name. Note Warp labels these `too-big` in Migration HQ
  and they cannot be automatically migrated — remediation is required before migration.
- Repositories 10 GB – < 40 GB (flagged `too-big` by Warp): list if any. Within the 40 GB
  hard limit but elevated risk — individual review recommended.
- Git LFS: note that Git LFS migration is not currently supported by Warp for either ADO or
  BBS sources. Repositories using LFS require manual handling. LFS usage cannot be determined
  from issue data alone — an explicit audit is recommended for repositories with large sizes.

---

### SECTION 6 — Warp Limitations and Considerations

**Technical Size Limits**

| Constraint | Limit | Impact if Exceeded |
|---|---|---|
| Maximum repository size | 40 GB | Cannot be automatically migrated |
| Maximum single file size | 400 MB | File blocks migration; must be removed |
| Maximum single commit size | 2 GB | Commit blocks migration; history rewrite required |
| Maximum Git reference length | 255 bytes | Long branch/tag names must be shortened |
| Repository metadata size | 20 GB | Metadata cannot exceed this limit |
| Git LFS support | Not supported (on roadmap) | LFS objects require manual handling |

**What Warp Migrates — Azure DevOps** *(include only if ADO repos are present)*

*Migrated from Azure DevOps:*
- Repository contents and Git history
- Pull requests, including: user history, work item links, and attachments
- Branch policies (not including user-scoped or cross-repository policies)

*Not migrated from Azure DevOps:*
- User-scoped branch policies
- Cross-repository branch policies
- Git LFS objects

**What Warp Migrates — Bitbucket Server** *(include only if BBS repos are present)*

*Migrated from Bitbucket Server:*
- Repository contents and Git history
- Pull requests, including: comments, reviews, review comments (file and line level),
  required reviewers, and attachments

*Not migrated from Bitbucket Server:*
- Personal repositories owned by individual users
- Branch permissions
- Commit comments
- Repository settings
- CI pipelines (Bamboo)

---

### SECTION 7 — Risk Assessment

A risk register derived from the actual inventory data and Warp's documented constraints. Likelihood and Impact must be
justified by real counts — do not use "unknown" for risks where the inventory provides
enough data to make an assessment.

**Universal Risks** (all inventories)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Repositories at or exceeding the 40 GB Warp size limit | Based on count from inventory | High | Identify remediation path before migration begins |
| High-activity repositories causing team disruption during cutover | Based on count with > 100 commits/year | Medium–High | Schedule during low-activity windows; communicate cutover dates |
| Git LFS dependencies undetectable from issue data | Cannot determine without explicit audit | High if present | Audit large repositories for LFS usage before migration begins |
| Credential expiry or misconfiguration blocking Warp | Pre-migration risk | High | Validate Warp Vault and test connectivity before each wave |

**Azure DevOps Specific Risks** *(include only if ADO repos are present)*

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Repositories with > 10 pipelines requiring manual CI/CD assessment | Based on count from inventory | Medium | Conduct CI/CD audit for each affected repository prior to migration |
| Azure Pipelines service connection not configured before rewiring | Pre-migration | High | Add service connection ID to config/warp.yml before Wave 1 begins |

**Bitbucket Server Specific Risks** *(include only if BBS repos are present)*

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| SFTP/SMB access to Bitbucket Server not configured in Warp Vault | Pre-migration | High | Configure and test SFTP (Linux) or SMB (Windows) access before migration |
| Intermediate storage bucket not configured or inaccessible | Pre-migration | High | Verify S3 or Azure Blob Storage configuration in Warp Vault |
| Bamboo CI pipelines have no migration path in Warp | Certain if Bamboo is in use | Medium | Plan separate CI migration for Bamboo pipelines before cutover |

Add additional rows for any specific risks visible in this inventory (e.g., a single
project dominating the backlog, large clusters of high-activity repos, etc.).

---

### SECTION 8 — Pre-Migration Checklist

Every item must be actionable by the customer's team without Packfiles staff involvement. Each bullet should be concise but contain enough detail to be directly actionable and specific to this inventory.

#### Environment Setup
1. Ensure credentials from Warp Vault are valid and have necessary permissions for all source repositories
2. Test connectivity to all source repositories using Warp's connectivity test tools
3. Confirm all expected repositories appear as open issues in Migration HQ

#### Repository Remediation
4. Identify all repositories flagged `too-big` in Migration HQ; confirm whether they are
   within the 40 GB hard limit or require remediation before migration
5. Identify any individual files > 400 MB and plan removal before migration
6. Audit large repositories for Git LFS usage; plan manual handling for any LFS dependencies
7. *(ADO only)* Review and document the CI/CD migration plan for repositories with > 10 pipelines

#### Destination Environment
8. Confirm the GitHub Enterprise Cloud organization is provisioned and licensed
9. Define the GitHub team structure and map source permissions to GitHub roles
10. Enable required GitHub features (Actions, rulesets, secret scanning, etc.)
11. Establish and document repository naming conventions for the destination

#### Migration HQ Configuration
12. Review and update `config/warp.yml` as needed:
    - Set `target_repo_visibility` (`"private"` or `"internal"`)
    - Define access policies
    - *(ADO only)* Add Azure Pipelines GitHub App service connection ID if rewiring is needed. Reference: https://kb.packfiles.io/using-warp/migration-hq/warp.yml

#### Communication and Coordination
13. Notify team leads of wave assignments, schedules, and expected cutover dates
14. Define and communicate the process for reporting migration issues
15. Document rollback criteria and procedures for failed migrations

---

### SECTION 9 — Post-Migration Actions

The following Warp slash commands are available for post-migration tasks. All slash commands are issued as **comments on the repository's backlog issue in Migration
HQ**. They cannot be invoked through Copilot conversation directly. Each command targets the specific repository represented by the issue on which it is posted.

**Available Universal Warp Slash Commands** *(all providers)*

| Slash Command | Purpose | When to Use |
|---|---|---|
| `/migrate` | Initiates the repository migration to GitHub | When ready to migrate a repository |
| `/rename-destination` | Renames the destination repository on GitHub | If the destination name needs to differ from the source |
| `/add-team` | Adds a GitHub team with specified permissions | After migration, to configure access controls |
| `/refresh` | Refreshes the backlog issue with the latest state | To update migration status |

**Azure DevOps Specific Warp Slash Commands** *(include only if ADO repos are present)*

Before using `/rewire-pipeline` or `/rewire-all-pipelines`, ensure the Azure Pipelines
GitHub App service connection ID is configured in `config/warp.yml` (Section 8, step 12).

| Slash Command | Purpose | When to Use |
|---|---|---|
| `/rewire-pipeline` | Rewires a specific Azure Pipeline to the migrated GitHub repository | After migration, per pipeline needing reconnection |
| `/rewire-all-pipelines` | Rewires all Azure Pipelines for the repository at once | After migration, when updating all pipelines together |
| `/integrate-boards` | Links Azure Boards to the migrated GitHub repository | If retaining Azure Boards for work item tracking |
| `/autolink-work-items` | Configures autolinks between GitHub and Azure Boards work items | To maintain traceability between commits and work items |
| `/lock-ado-repo` | Sets the source ADO repository to read-only | After successful migration |
| `/disable-ado-repo` | Disables the source ADO repository | When fully decommissioning the source |

---

### SECTION 10 — Licensing and Capacity Planning

A concise, yet professional, customer-facing section covering licensing options and cost estimates for this
migration. 

**Capacity Options**

| Tier | Repository Capacity | Price | Cost Per Repository |
|---|---|---|---|
| Free Trial | Up to 25 | $0 (no credit card required) | $0 |
| Small Box | 100 | $2,000 USD | $20 |
| Medium Box | 1,000 | $15,000 USD | $15 |
| Custom | 1,000+ | Contact Packfiles | — |

**Capacity Recommendation**

Based on the [N] total repositories in this inventory:
- Calculate the number of boxes of each type required and the total cost for each option. Only surface in the output the most cost-efficient option that meets the capacity needs of this migration and a brief explanation of the math. Remember to take the trial into account up to the trial repository capacity.
- State which option is most cost-efficient for this engagement (quantity of each box type and total cost)

Note: Capacity is only consumed on first successful distinct migration. Retries are not billed. Purchased Capacity associated with a Project does not expire.

For custom pricing on large engagements, contact Packfiles at https://packfiles.io/pricing.

---

### SECTION 11 — Key Resources

| Resource | URL |
|---|---|
| Packfiles Warp (GitHub Marketplace) | https://github.com/apps/packfiles-warp |
| Packfiles Knowledge Base | https://kb.packfiles.io |
| Warp Control Plane | https://warp.packfiles.io |
| Warp Quickstart Guide | https://kb.packfiles.io/guides/quickstart |
