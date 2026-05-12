# AWS EC2 Secure Deployment Architecture

## Overview

This project documents a secure EC2 deployment workflow focused on three operational priorities: **controlled access**, **baseline hardening**, and **support handoff readiness**.

The deployment follows a principle-first approach: restrict ingress before anything else, disable insecure authentication methods, apply a minimal OS baseline, and capture validation notes in a form a colleague or support reviewer can follow without prior context.

This is designed to reflect the operational mindset required for cloud support, AWS support associate, and junior DevOps roles — where understanding *why* a security control exists matters as much as knowing *how* to apply it.

---

## Architecture Flow

```
Local Admin Machine
        |
        | AWS CLI / AWS Console
        v
AWS IAM Permissions
  (EC2 launch, key pair, security group scoped to least privilege)
        |
        v
EC2 Instance (Amazon Linux / Ubuntu)
  (t2.micro or t3.micro — free tier eligible for lab)
        |
        v
Security Group
  (Inbound: SSH port 22 restricted to approved IP only)
  (Outbound: default allow — restrict further for production)
        |
        v
SSH Key Pair
  (PubkeyAuthentication only — password auth disabled)
  (.pem stored locally — never committed to source control)
        |
        v
Linux Baseline Hardening
  (sshd_config: PermitRootLogin no, MaxAuthTries 3, X11Forwarding no)
  (OS packages updated, unnecessary services disabled)
        |
        v
Validation + Operational Notes
  (SSH access verified, security group reviewed, baseline documented)
  (CI workflow validates repo hygiene on every push)
```

---

## Component Breakdown

| Component | Purpose | Security Consideration | Support Relevance |
|---|---|---|---|
| Local Admin Machine | Issues AWS CLI commands and SSH connections | Must not store credentials insecurely; use IAM roles or credential profiles | First point of misconfiguration; support often starts troubleshooting here |
| AWS IAM | Controls permissions to launch EC2, create key pairs, and modify security groups | Least privilege — only grant what is needed for this deployment | IAM policy errors are a top cause of support escalations |
| EC2 Instance | Hosts the deployed Linux environment | Patched regularly; no unnecessary services running | Instance state, type, and region must be understood for support triage |
| Security Group | Acts as a virtual firewall — controls inbound and outbound traffic at the instance level | SSH restricted to known IP; no open 0.0.0.0/0 for port 22 | Most common cause of SSH connectivity issues in AWS support tickets |
| SSH Key Pair | Authenticates the admin to the EC2 instance without a password | Private key stays local — never in source control or shared over insecure channels | Lost or wrong key file is a frequent user error requiring instance recovery steps |
| Linux Operating System | Provides the deployment environment | Regular patching; minimal installed software; sshd hardened | Linux command-line fluency is expected in cloud support roles |
| Validation Scripts | `scripts/setup.sh` — OS update and baseline service setup | Script syntax validated by CI; no credentials embedded | Automation that fails silently creates harder-to-diagnose support issues |
| Documentation / Runbook Notes | README, VALIDATION_LOG, TROUBLESHOOTING | Clear enough that another operator can reproduce the setup from cold | Documentation quality is a direct signal of support handoff discipline |

---

## Security Design Principles

### Least Privilege
IAM permissions are scoped to only what is needed for this deployment — EC2 launch, key pair management, and security group modification. Admin-level access is not assumed.

### Restricted Ingress
The security group allows SSH (port 22) only from a specific, approved IP address. Allowing 0.0.0.0/0 is avoided except in clearly documented temporary lab contexts.

### No Secrets Committed
No `.pem` files, `.env` files, AWS access keys, or private key content are committed to the repository. `.gitignore` and the CI secret hygiene check both enforce this.

### SSH Key Protection
The private key file is never stored in the repository, never shared over insecure channels, and file permissions are set to `chmod 400` before use.

### Patch Awareness
`scripts/setup.sh` runs OS package updates as the first step. Instance patching is documented as an ongoing responsibility, not a one-time action.

### Operational Validation
Each deployment step is validated before moving to the next. SSH access is confirmed. Security group rules are reviewed. Results are logged in `docs/VALIDATION_LOG.md`.

### Reproducible Documentation
The README, architecture doc, and validation log are written so that another operator can reproduce the setup or diagnose an issue without needing to contact the original author.

---

## Recruiter and Hiring Manager Signal

This architecture demonstrates:

| Signal | What it proves |
|---|---|
| EC2 fundamentals | Instance launch, AMI selection, key pair management, public IP handling |
| Linux support basics | SSH, sshd_config hardening, package management, service control |
| Cloud security awareness | Security group least-privilege, key-only auth, secret hygiene in source control |
| Documentation discipline | Architecture documented, validation logged, troubleshooting guide written |
| Incident-prevention mindset | Validation before deployment, CI hygiene checks, clear escalation notes |

The goal is to demonstrate the thinking that prevents common cloud support issues before they occur — and the documentation discipline that makes them faster to resolve when they do.
