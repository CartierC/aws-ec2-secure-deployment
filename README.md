# AWS EC2 Secure Deployment

**Secure AWS EC2 deployment lab documenting Linux instance provisioning, SSH hardening, security group controls, and operational validation — built as a cloud support portfolio project.**

[![Repo Validation](https://github.com/CartierC/aws-ec2-secure-deployment/actions/workflows/validate.yml/badge.svg)](https://github.com/CartierC/aws-ec2-secure-deployment/actions/workflows/validate.yml)

---

## Purpose

This repo documents a practical, end-to-end AWS EC2 support workflow:

- Launch a Linux EC2 instance with a restricted security group
- Configure SSH key-based access (no password authentication)
- Apply baseline server hardening using a reference `sshd_config`
- Validate access controls and document the operational baseline
- Produce clear, handoff-ready notes for cloud support review

This is a structured lab demonstrating the operational thinking required for cloud support, AWS support, and junior DevOps roles.

---

## Role Alignment

**Target roles:**

- Cloud Support Engineer
- AWS Support Associate
- Junior Cloud Engineer
- Technical Operations
- DevOps Support
- Cloud Security Support

This project demonstrates that a candidate understands secure access patterns, basic cloud operations, SSH/security group troubleshooting, and documentation discipline — the core signals for support and infrastructure roles.

---

## Architecture Overview

```
Local Admin Machine
        │
        ▼
AWS IAM / AWS CLI
        │
        ▼
EC2 Instance (Amazon Linux / Ubuntu)
        │
        ├── Security Group
        │     └── Inbound: SSH (port 22) restricted to trusted IP only
        │
        ├── SSH Key Pair
        │     └── PubkeyAuthentication only — no password login
        │
        └── Baseline Linux Hardening
              ├── sshd_config: PermitRootLogin no, MaxAuthTries 3
              ├── OS packages updated
              └── Validation / Operational Notes captured
```

---

## Folder Structure

```text
aws-ec2-secure-deployment/
├── README.md                        ← Project overview and usage guide
├── CONTRIBUTING.md                  ← Branching model and PR rules
├── .gitignore                       ← Excludes .pem, .env, secrets
├── configs/
│   └── sshd_config                  ← Hardened SSH daemon config reference
├── scripts/
│   └── setup.sh                     ← Baseline instance setup script
└── .github/
    └── workflows/
        └── validate.yml             ← CI: structure, syntax, and secret hygiene checks
```

---

## Prerequisites

- AWS account with access to EC2
- AWS CLI configured (`aws configure`) with IAM permissions for:
  - EC2 instance launch and termination
  - Security group create/modify
  - Key pair create
- An SSH key pair (`.pem` file — never commit this)
- Linux or macOS terminal
- Basic understanding of EC2, security groups, and SSH

---

## Usage

**1. Clone the repo for review:**

```bash
git clone https://github.com/CartierC/aws-ec2-secure-deployment.git
cd aws-ec2-secure-deployment
```

**2. Validate shell script syntax locally:**

```bash
bash -n scripts/setup.sh
```

**3. Make scripts executable before running:**

```bash
chmod +x scripts/*.sh
```

**4. Apply the SSH hardening config to an EC2 instance:**

```bash
# Copy reference config to the instance (adjust key path and IP)
scp -i path/to/key.pem configs/sshd_config ec2-user@PUBLIC_IP:/tmp/sshd_config

# On the instance: review and apply
sudo cp /tmp/sshd_config /etc/ssh/sshd_config
sudo sshd -t   # test config before restarting
sudo systemctl restart sshd
```

**5. Connect to the instance via SSH:**

```bash
ssh -i path/to/key.pem ec2-user@PUBLIC_IP
```

> **Never commit your `.pem` file.** It is excluded by `.gitignore` and the CI workflow will fail if one is detected.

---

## Security Controls Checklist

- [x] SSH access restricted — inbound port 22 limited to trusted IP in security group
- [x] No private keys committed — `.gitignore` excludes `.pem` and `.key` files; CI enforces this
- [x] Security group ingress minimized — no 0.0.0.0/0 SSH exposure
- [x] IAM least privilege considered — EC2 IAM role and key usage documented
- [x] Instance patching documented — OS package update step in `scripts/setup.sh`
- [x] Baseline Linux hardening applied — reference `sshd_config` with root login disabled, key auth only
- [x] Validation notes captured — CI workflow validates structure, scripts, and secret hygiene

---

## Why This Project Matters

This repo demonstrates:

- **AWS EC2 operational understanding** — instance provisioning, security group configuration, key pair management
- **Linux administration basics** — SSH daemon hardening, package management, service control
- **Cloud security awareness** — least-privilege access, secret hygiene, no credentials in source control
- **GitHub workflow discipline** — protected branches, CI validation, conventional commits, PR-based merge flow
- **Documentation for support handoff** — clear runbook-style notes a colleague or reviewer can follow without prior context

It does **not** claim production AWS administration experience. It is a structured lab proving readiness for junior cloud, technical support, and infrastructure operations roles.

---

## CI Validation

The `.github/workflows/validate.yml` workflow runs on every push and pull request to `main` and `develop`.

It checks:

| Check | What it validates |
|---|---|
| Repo structure | `README.md`, `configs/`, and `scripts/` all exist |
| Shell script syntax | `bash -n` and ShellCheck on all `scripts/*.sh` files |
| YAML validation | Python-based YAML parse of any `configs/*.yml` files |
| Secret hygiene | No `.pem`, `.env`, private key markers, or AWS key patterns found |

---

## Evidence and Validation

The following documents provide architecture reasoning, operational validation records, troubleshooting workflows, and a plan for visual evidence:

| Document | Purpose |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Full architecture breakdown with component table and security design principles |
| [docs/VALIDATION_LOG.md](docs/VALIDATION_LOG.md) | Validation checklist, local commands, CI summary, and result log template |
| [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | SSH failures, security group issues, script/YAML errors, secret hygiene fixes, escalation notes |
| [screenshots/README.md](screenshots/README.md) | Screenshot evidence plan — what to capture, what to redact, where to store |

---

## Next Improvements

- Add screenshots of EC2 instance settings with sensitive data redacted
- Add security group before/after comparison notes
- Add CloudWatch basic monitoring notes
- Add a teardown checklist to avoid unnecessary AWS costs
- Add IAM policy example for least-privilege EC2 access

---

## Recruiter Note

This repo is meant to be reviewed quickly. The core signal is not visual polish — it is proof of AWS, Linux, SSH, security group, and infrastructure documentation discipline, combined with a professional Git workflow.

**AWS Cloud Practitioner:** in progress.
