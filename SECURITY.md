# Security

## Branch Protection Required

Enable branch protection on `main` before production use. Without it, anyone
with write access can modify config and trigger syncs.

## Architecture

1. **PAT scope**: Token scoped to owner, `Secrets: Read/write` only
2. **Explicit allowlists**: Every repo must be listed in config
3. **Branch protection**: Require PR reviews for config changes

## Setup Branch Protection

Settings → Branches → Add rule:

- Pattern: `main`
- Require PR reviews
- Require approvals: 1

## CODEOWNERS

```text
/secrets-config.yml @your-username
/.github/workflows/sync-secrets.yml @your-username
```

## Limitations

Does NOT protect against:

- Compromised GitHub account
- Malicious collaborators with write access
- Workflow log exposure
- Secret values in git history
