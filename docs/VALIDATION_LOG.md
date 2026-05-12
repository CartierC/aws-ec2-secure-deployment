# Validation Log

This log documents the validation checks applied to this repository and the EC2 deployment workflow it describes. It serves as both an operational record and a reference for reviewers assessing the project's completeness.

---

## Validation Scope

This log covers validation of:

- Repository structure (required files and directories)
- Shell scripts (syntax and quality)
- Configuration files (YAML parsing, SSH baseline review)
- Secret hygiene (no committed credentials or private keys)
- SSH access assumptions (key-only auth, restricted security group)
- Security group assumptions (minimized ingress, no open 0.0.0.0/0 for SSH)
- Documentation completeness (README, architecture, troubleshooting, CONTRIBUTING)

---

## Validation Checklist

**Repository Structure**
- [x] `README.md` exists and explains the project purpose
- [x] `configs/` directory exists with SSH hardening reference
- [x] `scripts/` directory exists with setup script
- [x] `.github/workflows/validate.yml` CI workflow exists
- [x] `.gitignore` excludes `.pem`, `.env`, `.key`, and Terraform state files
- [x] `CONTRIBUTING.md` documents branching model and PR process
- [x] `docs/` directory contains architecture, validation, and troubleshooting docs

**Secret Hygiene**
- [x] No `.pem` files committed to repository
- [x] No `.env` files committed to repository
- [x] No private key markers (`BEGIN RSA PRIVATE KEY`, etc.) in any file
- [x] No AWS Access Key ID patterns (`AKIA...`) in any file
- [x] No hardcoded AWS secret key values detected

**Scripts**
- [x] `scripts/setup.sh` passes `bash -n` syntax validation
- [x] `scripts/setup.sh` contains no embedded credentials

**Configurations**
- [x] `configs/sshd_config` reviewed — `PermitRootLogin no`, key-only auth, `MaxAuthTries 3`
- [x] No YAML config files in `configs/` that require parsing (not applicable currently)

**CI Workflow**
- [x] `.github/workflows/validate.yml` is syntactically valid YAML
- [x] Workflow triggers on push and PR to `main` and `develop`
- [x] All four validation jobs present: structure, scripts, YAML, secret hygiene

**Documentation**
- [x] Branch protection guidance documented in `docs/BRANCH_PROTECTION.md`
- [x] Architecture documented in `docs/ARCHITECTURE.md`
- [x] Troubleshooting guide present in `docs/TROUBLESHOOTING.md`

---

## Local Validation Commands

Run these locally to replicate what the CI workflow checks. All commands are safe to run against the local clone.

**Check working tree is clean:**
```bash
git status
```
*Expected: no uncommitted or untracked sensitive files.*

**Scan for committed .pem files:**
```bash
find . -name "*.pem"
```
*Expected: no output. Any `.pem` file found in the working tree should be removed and added to `.gitignore`.*

**Scan for committed .env files:**
```bash
find . -name ".env" -o -name ".env.*"
```
*Expected: no output.*

**Scan for private key markers in file content:**
```bash
grep -R "BEGIN .*PRIVATE KEY" .
```
*Expected: no output. If found, the file must be removed from the repo and the key must be rotated immediately if it was a real credential.*

**Scan for AWS Access Key ID patterns:**
```bash
grep -R "AKIA[0-9A-Z]" .
```
*Expected: no output. AWS access keys must never appear in source-controlled files.*

**Validate shell script syntax:**
```bash
bash -n scripts/setup.sh
```
*Expected: no output (silent pass). Any output indicates a syntax error that must be fixed before the CI workflow will pass.*

**Validate shell script quality (if ShellCheck is installed):**
```bash
shellcheck scripts/setup.sh
```
*Expected: no warnings. ShellCheck is pre-installed on GitHub Actions ubuntu-latest runners.*

**Check SSH hardening config for key settings:**
```bash
grep -E "PermitRootLogin|PasswordAuthentication|MaxAuthTries" configs/sshd_config
```
*Expected: `PermitRootLogin no`, `PasswordAuthentication no`, `MaxAuthTries 3`.*

---

## CI Validation Summary

The `.github/workflows/validate.yml` workflow runs automatically on every push and pull request to `main` and `develop`. It performs four validation jobs:

| Job | What it checks |
|---|---|
| Repository structure | `README.md`, `configs/`, and `scripts/` all exist |
| Shell script syntax | `bash -n` on all `scripts/*.sh` files; ShellCheck if available |
| YAML validation | Python `yaml.safe_load()` on all `configs/*.yml` and `configs/*.yaml` files |
| Secret hygiene | `.pem` files, `.env` files, private key markers, and AWS access key patterns |

The workflow prints a summary on every run. A failed job blocks the PR merge when branch protection is configured.

---

## Manual AWS Validation Steps

These are the checks an operator should perform after deploying the EC2 instance described in this project. All examples use placeholder values — substitute your actual instance details.

**1. Confirm EC2 instance is in running state:**
```bash
aws ec2 describe-instances \
  --instance-ids i-PLACEHOLDER \
  --query 'Reservations[].Instances[].State.Name' \
  --output text
```
*Expected output: `running`*

**2. Confirm security group inbound rules:**
```bash
aws ec2 describe-security-groups \
  --group-ids sg-PLACEHOLDER \
  --query 'SecurityGroups[].IpPermissions'
```
*Expected: Only port 22 open, restricted to your approved IP — not `0.0.0.0/0`.*

**3. Confirm SSH access from approved IP:**
```bash
ssh -i path/to/key.pem ec2-user@PUBLIC_IP_PLACEHOLDER
```
*Expected: Successful connection. If it fails, see `docs/TROUBLESHOOTING.md`.*

**4. Confirm unauthorized ports are not exposed:**
```bash
# Run from outside the VPC if possible, or use AWS Security Group review
aws ec2 describe-security-groups \
  --group-ids sg-PLACEHOLDER \
  --query 'SecurityGroups[].IpPermissions[?ToPort!=`22`]'
```
*Expected: No unexpected open ports.*

**5. Confirm no credentials are stored in the repo:**
```bash
find . -name "*.pem" -o -name ".env"
grep -R "AKIA[0-9A-Z]" .
```
*Expected: No output.*

---

## Validation Result Template

Use this table to record validation results for each review cycle.

| Date | Validator | Check | Result | Notes |
|---|---|---|---|---|
| 2026-05-12 | CartierC | Repo structure | PASS | README, configs/, scripts/ all present |
| 2026-05-12 | CartierC | Secret hygiene | PASS | No .pem, .env, or key patterns found |
| 2026-05-12 | CartierC | Shell syntax | PASS | `bash -n scripts/setup.sh` — no errors |
| 2026-05-12 | CartierC | CI workflow | PASS | validate.yml present and syntactically valid |
| 2026-05-12 | CartierC | sshd_config review | PASS | PermitRootLogin no, PasswordAuthentication no |
| YYYY-MM-DD | Reviewer | Manual AWS validation | PENDING | Add after live EC2 deployment |
| YYYY-MM-DD | Reviewer | Security group review | PENDING | Add after live EC2 deployment |
