# Setup Guide

Complete guide to setting up secrets-sync in your account.

## Prerequisites

- GitHub account with admin access to your repositories
- GitHub CLI (`gh`) installed locally
- Fine-grained Personal Access Token (created below)

## Creating the PAT

The `GH_PAT_SECRETS_SYNC_ACTION` token powers all sync operations.

### Step-by-step

1. Visit [github.com/settings/personal-access-tokens/new][pat]
2. Configure the token:
   - **Token name**: `secrets-sync-action`
   - **Expiration**: 90 days recommended (set calendar reminder)
   - **Resource owner**: Your GitHub username
   - **Repository access**: All repositories
   - **Permissions** (only these two):
     - **Secrets**: Read and write
     - **Metadata**: Read-only (automatically selected)
3. Click "Generate token"
4. Copy the token immediately (you won't see it again)

### Why these permissions?

- `Secrets: Read and write` - Required to create/update secrets in target repos
- `Metadata: Read-only` - Required to verify repository access

**⚠️ Security Note on "All repositories" access:**

"All repositories" means the PAT can write secrets to ANY repository you create
in the future. If you prefer tighter security, use "Only select repositories"
and manually add each target repo. However, you'll need to update the PAT
configuration every time you add a new repository to sync.

GitHub's API automatically blocks access to repos outside your ownership, even
with this token. The token CANNOT access repositories belonging to other users
or organizations unless you have explicit write access to them.

## Setting Secrets

After forking the repo, set the PAT and your actual secrets:

```bash
# The PAT (required for the workflow to run)
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <your-username>/secrets-sync

# Your actual secrets (examples)
gh secret set DEPLOY_TOKEN --repo <your-username>/secrets-sync
gh secret set API_KEY --repo <your-username>/secrets-sync
gh secret set DATABASE_URL --repo <your-username>/secrets-sync
```

**Important**: Only set secret *values* in this repo. The `secrets-config.yml`
file contains only secret *names* and target repos (never values).

## Testing the Setup

Run a dry-run to verify everything works:

```bash
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=true

# Watch the workflow
gh run watch

# Check logs
gh run view --log
```

The dry-run mode previews changes without actually writing secrets.

## Verifying Secrets Synced

After a production run, verify secrets reached target repos:

```bash
# List secrets in a target repo (shows names only, not values)
gh secret list --repo <your-username>/target-repo
```

## Secret Rotation

To rotate the `GH_PAT_SECRETS_SYNC_ACTION` token:

1. Create a new PAT (see "Creating the PAT" above)
2. Update the secret:

   ```bash
   gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <your-username>/secrets-sync
   ```

3. Test with a dry run:

   ```bash
   gh workflow run sync-secrets.yml \
     --repo <your-username>/secrets-sync \
     -f dry_run=true
   ```

4. Revoke the old PAT at
   [github.com/settings/personal-access-tokens][pat-settings]

Set a calendar reminder to rotate PATs before they expire.

## Next Steps

- [Configure your secrets](configuration.md)
- [Review security best practices](security.md)
- [Learn about fork-friendly features](forking.md)

[pat]: https://github.com/settings/personal-access-tokens/new
[pat-settings]: https://github.com/settings/personal-access-tokens
