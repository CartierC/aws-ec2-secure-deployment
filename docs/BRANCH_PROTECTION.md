# Branch Protection Configuration

This document records the recommended GitHub branch protection settings for this repository.  
These settings protect `main` from accidental or unreviewed changes.

---

## Recommended Settings for `main`

| Setting | Value |
|---|---|
| Require pull request before merging | Enabled |
| Required approvals | 1 (minimum) |
| Dismiss stale reviews when new commits are pushed | Enabled |
| Require status checks to pass before merging | Enabled |
| Required status check | `validate` (Repo Validation workflow) |
| Require branches to be up to date before merging | Enabled |
| Restrict direct pushes to `main` | Enabled |
| Allow force pushes | Disabled |
| Allow deletions | Disabled |

---

## How to Apply in GitHub UI

1. Navigate to the repository on GitHub.
2. Go to **Settings** → **Branches**.
3. Under **Branch protection rules**, click **Add rule**.
4. In the **Branch name pattern** field, enter: `main`
5. Enable each setting from the table above.
6. Under **Require status checks to pass before merging**, search for and select: `validate`
   - This requires the CI workflow to have run at least once before the check appears.
7. Click **Create** (or **Save changes**).

---

## Why This Matters

Branch protection prevents:
- Accidental direct commits that skip CI validation.
- Force pushes that rewrite commit history.
- Merges before secret hygiene and structure checks pass.

For a portfolio project, this demonstrates awareness of professional Git workflow discipline — a signal hiring managers look for in cloud support and DevOps candidates.

---

## Current Status

Branch protection must be applied manually via the GitHub UI or via a GitHub token with `repo` admin scope.  
The `gh` CLI can apply this if authenticated with sufficient permissions:

```bash
gh api repos/CartierC/aws-ec2-secure-deployment/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["validate"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1,"dismiss_stale_reviews":true}' \
  --field restrictions=null
```

> Note: Run this only after the CI workflow has executed at least once so GitHub recognizes the `validate` check name.
