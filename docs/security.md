# Security Guide

Security architecture and best practices for secrets-sync.

## Safety Architecture (3 Layers)

### 1. Fine-grained PAT Scope

The `GH_PAT_SECRETS_SYNC_ACTION` token is restricted by:

- **Owner scope**: Token only works for repos under your username/org
- **Permission limits**: Only `Secrets: Read/write` and `Metadata: Read-only`
- **API enforcement**: GitHub API blocks access to repos you don't own

Even if the token leaks, it cannot access repos outside your control.

### 2. Explicit Allowlists

- `secrets-config.yml` contains explicit repository names
- No regex patterns or wildcards
- Every repo must be explicitly listed
- Changes tracked in git history

### 3. Branch Protection (Recommended)

Protect the `main` branch to require reviews for critical file changes.

## Setting Up Branch Protection

1. Go to Settings → Branches → Add rule
2. Branch name pattern: `main`
3. Enable:
   - Require pull request reviews before merging
   - Require approvals: 1
   - Dismiss stale reviews when new commits are pushed
4. (Optional) Require status checks to pass

## CODEOWNERS Integration

Create `.github/CODEOWNERS` to require approval for security-sensitive files:

```text
# Require approval for critical files
/secrets-config.yml @your-username
/.github/workflows/sync-secrets.yml @your-username
```

Now changes to these files require your explicit approval before merge.

## What This Does NOT Protect Against

- **Compromised GitHub account**: If your account is compromised, an attacker
  can modify secrets directly
- **Malicious collaborators**: Anyone with write access can modify the config
- **Secret values in config**: Never put actual secret values in
  `secrets-config.yml` (only names and repos)

## Best Practices

### PAT Management

- **Expiration**: Set PATs to expire (90 days recommended)
- **Calendar reminders**: Set reminders to rotate before expiration
- **Immediate revocation**: If token leaks, revoke immediately and create new
- **Audit regularly**: Review token permissions quarterly

### Secret Hygiene

- **Unique secrets**: Don't reuse secrets across different services
- **Rotation schedule**: Rotate secrets on a regular schedule
- **Audit trail**: Track when secrets were last updated
- **Remove unused**: Clean up secrets that are no longer needed

### Repository Access

- **Principle of least privilege**: Only sync secrets to repos that need them
- **Regular audits**: Review repository lists quarterly
- **Remove inactive repos**: Remove repos that are archived or no longer active

### Monitoring

Monitor workflow runs for anomalies:

```bash
# Check recent runs
gh run list --repo <your-username>/secrets-sync --workflow=sync-secrets.yml

# View logs for suspicious activity
gh run view <run-id> --log
```

Look for:

- Unexpected repositories being synced
- Failed syncs to previously working repos
- Changes to secrets-config.yml from unknown sources

## Incident Response

If you suspect a security incident:

1. **Immediately revoke** the `GH_PAT_SECRETS_SYNC_ACTION` token
2. **Rotate all synced secrets** in target repositories
3. **Review git history** for unauthorized changes
4. **Check workflow logs** for suspicious activity
5. **Create new PAT** with fresh permissions
6. **Update secret values** before re-enabling sync

## Security Limitations

This tool does **not**:

- Automatically rotate secret values
- Detect compromised secrets
- Encrypt secret values (GitHub handles this)
- Prevent secrets from being exposed in workflow logs
- Audit secret usage in target repos

## Next Steps

- [Review troubleshooting guide](troubleshooting.md)
- [Learn about forking](forking.md)
