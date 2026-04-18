# Setup

## Create PAT

1. [github.com/settings/personal-access-tokens/new][pat]
2. Configure:
   - Name: `secrets-sync-action`
   - Expiration: 90 days
   - Repository access: Select target repos
   - Permissions: `Secrets: Read/write`, `Variables: Read/write`, `Metadata: Read-only`
3. Copy token

## Set Secrets

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <user>/secrets-sync
gh secret set YOUR_SECRET --repo <user>/secrets-sync
```

## Set Variables

Variable values are stored on the secrets-sync repo itself, then pushed to targets by the workflow.

```bash
gh variable set YOUR_VARIABLE --repo <user>/secrets-sync --body "value"
```

## Test

```bash
# Dry run
gh workflow run sync-secrets.yml --repo <user>/secrets-sync -f dry_run=true

# Check
gh run watch
```

[pat]: https://github.com/settings/personal-access-tokens/new
