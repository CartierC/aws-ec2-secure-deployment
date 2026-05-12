# Contributing Guide

This document defines the branching model, pull request process, and commit style for this repository.

---

## Branching Model

| Branch | Purpose |
|---|---|
| `main` | Stable portfolio release branch. Protected. |
| `develop` | Active integration branch for reviewed enhancements. |
| `feature/*` | Scoped improvements branched from `main` or `develop`. |

**Branch lifecycle:**

```
main
 └── feature/your-improvement    ← branch from main
       └── (work, commit, push)
             └── PR → develop     ← merge into develop first
                   └── PR → main  ← merge into main after CI passes
```

Do not create hotfix branches unless there is an actual production bug or urgent patch.

Evidence and validation branches should use the naming convention `feature/validation-evidence`. Documentation-only updates should include a brief validation note in the PR body confirming no secrets were introduced and any scripts still pass `bash -n`.

---

## Pull Request Rules

- Feature branches must merge into `develop` first, not directly into `main`.
- `main` only receives stable, CI-validated changes from `develop`.
- Avoid direct pushes to `main` — branch protection should enforce this.
- Every PR should include:
  - **Purpose:** What this change does and why.
  - **Files changed:** A brief list of what was added or modified.
  - **Validation steps:** How you confirmed the change is correct (CI pass, bash -n, manual review, etc.).

---

## Commit Message Style

Follow the conventional commit prefix pattern:

| Prefix | Use for |
|---|---|
| `docs:` | README, runbooks, documentation updates |
| `ci:` | GitHub Actions workflows, validation scripts |
| `security:` | Security controls, secret hygiene, hardening configs |
| `feat:` | New features or functionality |
| `fix:` | Bug fixes |
| `chore:` | Maintenance, dependency updates, project structure |
| `refactor:` | Code restructuring without behavior change |

**Examples:**

```
docs: improve EC2 deployment README with architecture overview
ci: add repository structure and secret hygiene validation workflow
security: add hardened sshd_config baseline reference
chore: update .gitignore to exclude .pem and .env files
```

---

## Secret Hygiene

Never commit:
- AWS credentials or access keys
- `.pem` private key files
- `.env` or secret configuration files
- SSH private keys of any kind

The CI validation workflow will fail on any of the above. Review `.gitignore` before staging changes.

---

## Reviewer Notes

This repo serves as a cloud support and infrastructure portfolio project. Changes should improve recruiter-readability, operational clarity, or security posture — not add complexity for its own sake.
