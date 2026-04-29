# AWS EC2 Secure Deployment

Secure AWS EC2 deployment lab with hardened instance setup, baseline server configuration, SSH controls, and cloud support documentation.

## Purpose

This repo documents a practical AWS EC2 support workflow: launch a Linux instance, restrict access, apply baseline hardening, validate SSH/security group behavior, and leave clear operational notes for review.

## Role Alignment

**Target roles:** Cloud Support Engineer, AWS Support Associate, Junior Cloud Engineer, Technical Operations

This project is positioned for support and infrastructure roles where hiring managers need to see that the candidate understands secure access, basic cloud operations, and troubleshooting documentation.

## Skills Demonstrated

- AWS EC2 instance deployment concepts
- Linux server access and baseline configuration
- SSH key-based access controls
- Security group review and least-access thinking
- Cloud support documentation
- Infrastructure troubleshooting workflow
- Operational hygiene and repeatable notes

## Tech Stack

- AWS EC2
- Linux / Ubuntu concepts
- SSH
- Security Groups
- Bash documentation patterns
- Markdown runbooks

## Project Structure

```text
aws-ec2-secure-deployment/
├── README.md
├── docs/
│   └── runbook.md
├── configs/
│   └── security-baseline-notes.md
├── screenshots/
│   └── README.md
└── sample-output/
    └── ec2-validation-output.txt
```

Some folders may contain documentation placeholders until screenshots, command output, or AWS console evidence are added.

## How to Run or Review

This repo is documentation-first. A reviewer should inspect:

1. The deployment checklist
2. SSH/security group assumptions
3. Baseline hardening notes
4. Sample validation output
5. Next improvement list

Suggested review flow:

```bash
git clone https://github.com/CartierC/aws-ec2-secure-deployment.git
cd aws-ec2-secure-deployment
cat README.md
cat sample-output/ec2-validation-output.txt
```

## Sample Output

```text
EC2 Validation Summary
Instance access method: SSH key authentication
Inbound access reviewed: SSH restricted to trusted source IP
OS baseline reviewed: package updates, user access, firewall notes
Status: Documentation-ready lab for cloud support review
```

Full example: `sample-output/ec2-validation-output.txt`

## What This Project Proves

This project proves practical cloud support thinking: secure access first, document the environment clearly, validate the baseline, and make the work understandable for another operator.

It does **not** claim production AWS administration experience. It is a structured lab used to show readiness for junior cloud, technical support, and infrastructure operations roles.

## Next Improvements

- Add screenshots of EC2 instance settings with sensitive data hidden
- Add security group before/after notes
- Add CloudWatch monitoring notes
- Add a teardown checklist to avoid unnecessary AWS costs
- Add a small Bash validation helper for local Linux checks

## Recruiter Note

This repo is meant to be reviewed quickly. The core signal is not visual polish; it is proof of AWS, Linux, SSH, and infrastructure documentation discipline.

**AWS Cloud Practitioner:** in progress.
