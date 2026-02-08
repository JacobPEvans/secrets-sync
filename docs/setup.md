# Setup

## Create PAT

1. [github.com/settings/personal-access-tokens/new][pat]
2. Configure:
   - Name: `secrets-sync-action`
   - Expiration: 90 days
   - Repository access: Select target repos
   - Permissions: `Secrets: Read/write`, `Metadata: Read-only`
3. Copy token

## Set Secrets

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <user>/secrets-sync
gh secret set YOUR_SECRET --repo <user>/secrets-sync
```

## Test

```bash
# Dry run
gh workflow run sync-secrets.yml --repo <user>/secrets-sync -f dry_run=true

# Check
gh run watch
```

[pat]: https://github.com/settings/personal-access-tokens/new
