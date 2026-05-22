# Scoring Rules for plan-migration

Loaded by `skills/plan-migration/SKILL.md` during Step 3. Apply these rules exactly as written.

---

## Size Score (0–5 points) — identical for ADO and BBS

| Repository Size | Points | Label |
|---|---|---|
| < 10 MB | 5 | Excellent |
| 10 MB – < 100 MB | 4 | Very Good |
| 100 MB – < 500 MB | 3 | Good |
| 500 MB – < 5 GB | 2 | Fair |
| 5 GB – < 40 GB | 1 | Poor |
| ≥ 40 GB | 0 | Exceeds Warp limit |

Size bands are also used for the Size Distribution table in the output document.

Sources:
- https://kb.packfiles.io/migrations/azure-devops/limitations
- https://kb.packfiles.io/migrations/bitbucket-server/limitations

> **Note:** Packfiles Warp's absolute repository size limit is **40 GB**. Repositories at or above
> this threshold cannot be migrated automatically and require manual remediation prior to migration.

---

## Activity Score (0–3 points) — identical for ADO and BBS

| Commits Past Year | Points | Label |
|---|---|---|
| 0 | 3 | Inactive |
| 1–10 | 2 | Low Activity |
| 11–100 | 1 | Moderate Activity |
| > 100 | 0 | High Activity |

---

## Simplicity Score (0–3 points) — differs by provider

### Azure DevOps — scored on PR count AND pipeline count

| Condition | Points | Label |
|---|---|---|
| 0 PRs AND ≤ 1 pipeline | 3 | Simple |
| ≤ 10 PRs AND ≤ 3 pipelines | 2 | Moderate |
| ≤ 50 PRs AND ≤ 10 pipelines | 1 | Complex |
| > 50 PRs OR > 10 pipelines | 0 | Very Complex |

### Bitbucket Server — scored on PR count only (no pipelines exist)

| Condition | Points | Label |
|---|---|---|
| 0 PRs | 3 | Simple |
| ≤ 10 PRs | 2 | Moderate |
| ≤ 50 PRs | 1 | Complex |
| > 50 PRs | 0 | Very Complex |

---

## Total Score and Wave Assignment

Sum the three component scores. Maximum possible score is 11.

| Total Score | Wave | Risk Level |
|---|---|---|
| 9–11 | Wave 1 | Low Risk |
| 6–8 | Wave 2 | Medium Risk |
| 3–5 | Wave 3 | High Risk |
| 0–2 | Manual Review | Very High Risk |

---

## Override Rules — Manual Review

These override the score-based assignment. Check overrides **after** scoring.

**Azure DevOps** — move to Manual Review regardless of score if:
- Repository size ≥ 40 GB, OR
- Pipeline count > 10

**Bitbucket Server** — move to Manual Review regardless of score if:
- Repository size ≥ 40 GB

BBS has no pipeline-count override. Do not apply ADO pipeline override logic to BBS repos.