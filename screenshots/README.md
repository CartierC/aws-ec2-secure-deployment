# Screenshots

This folder is reserved for recruiter-facing validation evidence — visual proof that the deployment workflow was executed, not just documented.

---

## Recommended Screenshots to Add

Capture these after performing the live EC2 deployment described in this project. All screenshots must have sensitive details redacted before committing.

| # | Screenshot | What to capture | Sensitive fields to blur |
|---|---|---|---|
| 1 | GitHub Actions — passing workflow | The `validate` workflow showing all checks green | None required |
| 2 | GitHub repository — branch structure | Branches list showing `main`, `develop`, `feature/*` | None required |
| 3 | GitHub — Release v1.0.0 | The release page showing tag, title, and notes | None required |
| 4 | AWS Console — EC2 instance running | Instance list or detail showing state: running | Account ID, Instance ID (partial OK), Public IP |
| 5 | AWS Console — Security group inbound rules | Rules showing port 22 restricted to approved IP, not 0.0.0.0/0 | Source IP, Security Group ID |
| 6 | Terminal — successful SSH connection | Terminal output after SSH login (command prompt on instance) | Public IP, username (optional), any internal hostnames |
| 7 | Terminal — bash -n validation | Local terminal showing `bash -n scripts/setup.sh` with no errors | None required |
| 8 | README preview | The GitHub-rendered README showing CI badge, sections, and structure | None required |

---

## Rules for All Screenshots

- **Blur or redact:** AWS Account ID, full public IP addresses, real instance IDs, internal hostnames, any personal or organizational information
- **Never show:** Private key content, AWS Access Key ID or Secret, passwords, internal VPC CIDR ranges if sensitive
- **Safe to show:** EC2 instance state (running/stopped), security group rule port numbers, terminal command output, repo structure, GitHub Actions results

---

## How to Add Screenshots

1. Capture the screenshot and save it to this folder.
2. Use a clear, descriptive filename: `github-actions-passing.png`, `ec2-instance-running.png`, etc.
3. Verify the image for sensitive content before committing.
4. Reference the screenshot in the README or validation log as needed.
5. Commit with: `docs: add validation screenshots`

---

## Note on Fabricated Evidence

Do not fabricate screenshots, paste in stock AWS console images, or claim results that were not actually achieved. Recruiters and technical interviewers can often identify doctored evidence, and honest documentation of a lab setup is more credible than a polished fake.

If the live deployment has not been performed yet, leave this folder as a placeholder and add the note `[PENDING: screenshots to be added after live deployment]` to the validation log.
