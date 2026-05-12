# Troubleshooting Guide

This guide covers the most common issues encountered during EC2 secure deployment, SSH access, script validation, and CI workflow failures. Each section follows the cloud support pattern: symptoms first, then likely causes, then diagnostic steps.

---

## SSH Connection Fails

### Symptoms
- `Permission denied (publickey)`
- `Connection timed out`
- `ssh: connect to host PUBLIC_IP port 22: Connection refused`
- `Host unreachable` or `No route to host`

### Possible Causes

| Cause | Likelihood |
|---|---|
| Wrong private key file specified | High |
| Key file permissions too open (`chmod` not set to 400) | High |
| Security group not allowing port 22 from your IP | High |
| Wrong username for the AMI (e.g., `ubuntu` vs `ec2-user`) | High |
| Instance is stopped or terminated | Medium |
| Instance public IP changed after stop/start | Medium |
| Network ACL or VPC routing blocking the connection | Low |

### Diagnostic Steps

**1. Fix key file permissions:**
```bash
chmod 400 path/to/key.pem
```
SSH clients reject keys that are readable by others. This is the most common error for new AWS users.

**2. Confirm the instance is running:**
```bash
aws ec2 describe-instances \
  --instance-ids i-PLACEHOLDER \
  --query 'Reservations[].Instances[].State.Name' \
  --output text
```
Expected: `running`. If stopped, start it first.

**3. Get the current public IP:**
```bash
aws ec2 describe-instances \
  --instance-ids i-PLACEHOLDER \
  --query 'Reservations[].Instances[].PublicIpAddress' \
  --output text
```
Public IPs change when an instance is stopped and restarted unless an Elastic IP is assigned.

**4. Confirm security group allows your IP on port 22:**
```bash
aws ec2 describe-security-groups \
  --group-ids sg-PLACEHOLDER \
  --query 'SecurityGroups[].IpPermissions[?ToPort==`22`]'
```
The inbound rule should list your current IP, not `0.0.0.0/0`.

**5. Confirm the correct username:**
| AMI | Default username |
|---|---|
| Amazon Linux 2 / 2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| RHEL | `ec2-user` or `root` |
| Debian | `admin` |
| CentOS | `centos` |

**6. Retry with verbose SSH output:**
```bash
ssh -v -i path/to/key.pem ec2-user@PUBLIC_IP_PLACEHOLDER
```
Look for `Authentications that can continue` and `Server accepts key` in the output.

---

## Security Group Misconfiguration

### Symptoms
- SSH is unavailable despite instance running
- Access works from an unexpected network or IP range
- Unexpected ports appear open in a port scan

### Fix Approach

**Limit SSH ingress to your known IP only:**
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-PLACEHOLDER \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_IP_PLACEHOLDER/32
```

**Remove an overly permissive rule (e.g., 0.0.0.0/0 on port 22):**
```bash
aws ec2 revoke-security-group-ingress \
  --group-id sg-PLACEHOLDER \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**Review all inbound rules:**
```bash
aws ec2 describe-security-groups \
  --group-ids sg-PLACEHOLDER \
  --query 'SecurityGroups[].IpPermissions'
```

**General rules:**
- SSH (port 22) should only be open to approved IPs — never `0.0.0.0/0` in anything beyond an isolated lab
- Clearly document any temporary 0.0.0.0/0 rules with a comment and a removal plan
- Remove all unused inbound rules as part of deployment cleanup

---

## Script Validation Fails

### Symptoms
- CI workflow fails on the "Validate shell scripts" step
- `bash -n scripts/setup.sh` returns an error locally
- Script is not executable when deployed to instance

### Diagnostic Steps

**Check syntax locally:**
```bash
bash -n scripts/setup.sh
```
Silent output means the syntax is valid. Any output describes the error and line number.

**Make scripts executable:**
```bash
chmod +x scripts/*.sh
```
Scripts that lack execute permission will fail with `Permission denied` when run directly.

**Run ShellCheck locally (if installed):**
```bash
shellcheck scripts/setup.sh
```
ShellCheck catches issues that `bash -n` misses, including unsafe variable expansion and quoting errors.

**Common shell script errors:**
| Error | Cause | Fix |
|---|---|---|
| `syntax error near unexpected token` | Missing `fi`, `done`, or `;;` | Review control flow structure |
| `command not found` | Path issue or typo | Check command name and PATH |
| Unquoted variable | Variable expansion with spaces | Quote variables: `"$VAR"` |
| `Permission denied` | Missing `+x` | `chmod +x scripts/*.sh` |

---

## YAML Validation Fails

### Symptoms
- CI fails on the "Validate YAML files" step
- Python reports a `yaml.YAMLError` in a config file
- Indentation errors visible in the workflow run log

### Diagnostic Steps

**Parse YAML locally:**
```bash
python3 -c "import yaml; yaml.safe_load(open('configs/your-file.yml'))"
```
Silent output means valid. Any output describes the error and line number.

**Common YAML errors:**
| Error | Cause | Fix |
|---|---|---|
| `mapping values are not allowed here` | Colon in an unquoted value | Quote the value: `key: "value: with colon"` |
| `found character that cannot start any token` | Tab character used for indentation | Replace all tabs with spaces |
| `expected <block end>` | Indentation inconsistency | Use a consistent 2-space indent |
| `could not find expected ':'` | Missing colon in key-value pair | Review the key-value structure |

**Check for tabs in YAML files:**
```bash
grep -P "\t" configs/*.yml
```
YAML does not allow tab characters for indentation.

---

## Secret Hygiene Failure

### Symptoms
- CI fails on the "Secret and key hygiene check" step
- CI output shows: `FAIL: .pem file(s) found in repository`
- CI output shows: `FAIL: AWS Access Key ID pattern detected`
- CI output shows: `FAIL: Private key markers found in file content`

### Fix Steps

**Remove the file from the working tree:**
```bash
git rm --cached path/to/secret-file.pem
```
This removes it from Git tracking without deleting the local file.

**Add it to .gitignore to prevent re-commit:**
```bash
echo "path/to/secret-file.pem" >> .gitignore
git add .gitignore
git commit -m "security: exclude accidental secret file from tracking"
```

**If the secret was a real AWS key or credential — rotate it immediately:**
1. Go to AWS IAM → Your user → Security credentials
2. Deactivate and delete the exposed access key
3. Create a new access key
4. Update any services or local configurations using the old key

**If the secret was committed in a previous commit:**
The file may still exist in Git history even after removal. Consult a senior team member or use `git filter-repo` to rewrite history — this is a destructive operation that requires care.

**Verify the fix before pushing:**
```bash
find . -name "*.pem"
grep -R "AKIA[0-9A-Z]" .
grep -R "BEGIN .*PRIVATE KEY" .
```
All three should return no output.

---

## Escalation Notes

Escalate to a senior engineer or security team when:

- **Suspected credential exposure** — a real AWS key, SSH private key, or password was committed and potentially accessed. Rotate immediately and treat as an incident.
- **Unknown network access to the instance** — unexpected connections in CloudWatch or VPC Flow Logs that cannot be explained by approved use.
- **Production instance impact** — this guide covers lab and portfolio deployments; any production system issue should follow your organization's incident response process.
- **Repeated SSH failure after security group and key checks** — may indicate a deeper networking issue (VPC ACL, route table, or instance profile) requiring AWS support.
- **CI consistently fails on a check that appears correct** — could be a runner environment issue; open an issue in the repository with the full CI log.

---

## Quick Reference

| Problem | First check | Command |
|---|---|---|
| SSH permission denied | Key file permissions | `chmod 400 key.pem` |
| SSH timeout | Security group port 22 rule | `aws ec2 describe-security-groups` |
| Wrong username | AMI type | See username table above |
| Instance unreachable | Instance state | `aws ec2 describe-instances` |
| CI script check fails | Syntax error | `bash -n scripts/*.sh` |
| CI YAML check fails | Indentation or tab | `python3 -c "import yaml; yaml.safe_load(open('file.yml'))"` |
| CI secret check fails | Committed key or credential | `find . -name "*.pem"` |
