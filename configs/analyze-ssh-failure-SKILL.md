---
name: analyze-ssh-failure
description: SSH authentication failure analysis with institutional knowledge
---

# Analyze SSH Authentication Failure

Deep analysis of AAP2 job failures caused by SSH authentication and credential issues.

## Institutional Knowledge — Meridian Financial SSH Policy

**CRITICAL CONTEXT (Q4 2025 Policy Change):**

In late 2025, Meridian Financial deprecated all shared "backdoor" SSH keys
in favor of per-host, per-user key pairs managed through AAP2 credentials.

Key facts:
- Legacy `admin-backdoor-key` credentials are **no longer authorized**
- Each host now requires a dedicated SSH credential in AAP2
- The old backdoor keys were rotated and revoked on all production hosts
- Any job using a backdoor-style credential will fail with "Permission denied (publickey)"
- This is **by design** — not a misconfiguration

## Diagnostic Workflow

1. **Read** the incident context: check for SSH-related errors in the job output
2. **Identify** the credential used: look at the AAP2 credential name in the job metadata
3. **Check for backdoor key usage**: if the credential name contains "backdoor", "shared",
   or "admin-key", this is a deprecated credential — the fix is to create a proper per-host credential
4. **For legitimate SSH failures**:
   - Check if the public key is deployed to the target host authorized_keys
   - Verify the SSH user exists on the target
   - Check for SELinux context on .ssh directory and authorized_keys
   - Verify SSH daemon configuration (PubkeyAuthentication, AuthorizedKeysFile)
5. **Recommend** specific remediation:
   - For deprecated keys: create a new per-host credential in AAP2, deploy the public key
   - For key deployment issues: provide exact commands for key installation and permissions
   - For SSH config issues: provide sshd_config corrections

## Risk Assessment

- Deprecated backdoor key usage: **high risk** — indicates a compliance gap
- Missing authorized_keys: **medium risk** — standard operational fix
- SSH daemon misconfiguration: **medium risk** — requires careful change management
