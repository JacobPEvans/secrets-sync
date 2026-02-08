# secrets-sync

[![Sync Secrets](https://github.com/JacobPEvans/secrets-sync/actions/workflows/sync-secrets.yml/badge.svg)](https://github.com/JacobPEvans/secrets-sync/actions/workflows/sync-secrets.yml)

Centralized GitHub Actions secret management for multi-repository projects.
Define secrets once in this repo, and they automatically sync to all your
target repositories.

## What is this?

If you maintain multiple GitHub repositories that share the same secrets
(API tokens, deployment keys, etc.), manually updating them is tedious and
error-prone. This repo automates that process using GitHub Actions.

**Instead of**: Manually updating `DEPLOY_TOKEN` in 20 repositories

**You do**: Update it once here, and it syncs everywhere automatically

## How it works

```text
┌─────────────────────────────────┐
│  secrets-sync Repository        │
│                                  │
│  ┌──────────────────────────┐  │
│  │ secrets-config.yml       │  │
│  │ - Which secrets          │  │
│  │ - Which repos            │  │
│  └──────────────────────────┘  │
│            ▼                     │
│  ┌──────────────────────────┐  │
│  │ GitHub Actions Workflow  │  │
│  │ (triggered on push/manual)│ │
│  └──────────────────────────┘  │
└─────────────┬───────────────────┘
              │
              ▼
    ┌─────────────────────┐
    │ GitHub API          │
    │ (with PAT auth)     │
    └─────────────────────┘
              │
      ┌───────┴───────┐
      ▼               ▼
┌──────────┐    ┌──────────┐
│ Repo A   │    │ Repo B   │
│ (updated)│    │ (updated)│
└──────────┘    └──────────┘
```

**Steps:**

1. You define secrets and target repos in `secrets-config.yml`
1. You store the secret values in this repo's GitHub secrets
1. Workflow runs (on push or manual trigger)
1. GitHub Actions API writes secrets to target repos
1. All repos now have the latest secret values

## Prerequisites

Before using this repo, you need:

- **GitHub account** with admin access to your repositories
- **Fine-grained Personal Access Token (PAT)** with these permissions:
  - Repository access: All repositories (or specific ones)
  - Permissions: `Secrets: Read and write`, `Metadata: Read-only`
- **GitHub CLI (`gh`)** installed locally (for setting secrets)

## Quick Start

### 1. Fork this repository

Click "Fork" at the top right of this page. The configuration automatically
adapts to your GitHub username.

### 2. Create a Personal Access Token

1. Go to [github.com/settings/personal-access-tokens/new][pat]
1. Configure:
   - **Name**: `secrets-sync-action`
   - **Resource owner**: Your username
   - **Repository access**: All repositories
   - **Permissions**:
     - `Secrets`: Read and write
     - `Metadata`: Read-only
1. Click "Generate token" and copy it immediately

### 3. Set the PAT in your fork

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION \
  --repo <your-username>/secrets-sync
```

When prompted, paste your PAT.

### 4. Configure your secrets

Edit `secrets-config.yml` to define which secrets go to which repos:

```yaml
secrets:
  - name: MY_API_TOKEN
    repositories:
      - my-app
      - my-other-app
      - __SELF__  # This means your profile repo (<username>/<username>)
```

### 5. Set your secret values

```bash
gh secret set MY_API_TOKEN --repo <your-username>/secrets-sync
```

### 6. Push and verify

```bash
git add secrets-config.yml
git commit -m "Configure secret distribution"
git push

# Watch the workflow run
gh run watch
```

The workflow automatically runs on push and syncs your secrets!

## Configuration

### Understanding `secrets-config.yml`

This YAML file defines the mapping between secrets and repositories:

```yaml
secrets:
  - name: SECRET_NAME          # Exact name of the secret
    # Comment: What this secret is for
    repositories:
      - repo-name-1            # Without owner prefix
      - repo-name-2
      - __SELF__               # Special marker for profile repo
```

**Key points:**

- Repository names are **without** the owner prefix (added automatically)
- `__SELF__` resolves to your profile repository (e.g.,
  `JacobPEvans/JacobPEvans`)
- Comments are supported (and encouraged!) to document each secret's purpose
- The config file itself contains no secret values (only names and target
  repos)

### Adding a new secret

1. Edit `secrets-config.yml`:

```yaml
secrets:
  - name: NEW_SECRET_NAME
    # Purpose of this secret
    repositories:
      - target-repo-1
      - target-repo-2
```

1. Set the secret value:

```bash
gh secret set NEW_SECRET_NAME --repo <your-username>/secrets-sync
```

1. Push the config change:

```bash
git add secrets-config.yml
git commit -m "Add NEW_SECRET_NAME to sync"
git push
```

The workflow runs automatically and syncs the new secret.

### Removing a repository from a secret

Edit `secrets-config.yml` and remove the repo from the list. The workflow does
**not** delete secrets from repos (by design - deletion would be destructive).
To fully remove:

1. Edit `secrets-config.yml` and push
1. Manually delete from the target repo:

```bash
gh secret remove SECRET_NAME --repo <your-username>/target-repo
```

## Setup

### Creating the required PAT

The `GH_PAT_SECRETS_SYNC_ACTION` token powers all sync operations. Here's how
to create it:

1. Visit [github.com/settings/personal-access-tokens/new][pat]
1. Token name: `secrets-sync-action`
1. Expiration: Choose based on your security policy (90 days recommended)
1. Resource owner: Your GitHub username
1. Repository access: All repositories
1. Permissions (only these two):
   - **Secrets**: Read and write
   - **Metadata**: Read-only (automatically selected)
1. Click "Generate token"
1. Copy the token immediately

**Why these specific permissions?**

- `Secrets: Read and write` - Required to create/update secrets in target
  repos
- `Metadata: Read-only` - Required to verify repository access

GitHub's API automatically blocks access to repos outside your ownership, even
with this token.

### Setting secrets

Set the PAT and your actual secrets in this repo:

```bash
# The PAT (required for the workflow)
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <your-username>/secrets-sync

# Your actual secrets (examples)
gh secret set DEPLOY_TOKEN --repo <your-username>/secrets-sync
gh secret set API_KEY --repo <your-username>/secrets-sync
```

**Important**: Only set secret *values* in this repo. The `secrets-config.yml`
file only contains secret *names* and target repos.

## Usage

### Automatic triggers

The workflow runs automatically when:

- You push changes to `.github/workflows/sync-secrets.yml`
- You push changes to `secrets-config.yml`

Both trigger a full sync of all secrets.

### Manual trigger

Run the workflow manually with optional dry-run mode:

```bash
# Dry run (preview only, no changes)
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=true

# Production run (writes secrets)
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=false
```

### Checking workflow status

```bash
# List recent runs
gh run list --repo <your-username>/secrets-sync --workflow=sync-secrets.yml

# Watch the latest run
gh run watch

# View logs for a specific run
gh run view <run-id> --log
```

### Verifying secrets synced

Check if secrets reached target repos:

```bash
# List secrets in a target repo (names only, not values)
gh secret list --repo <your-username>/target-repo
```

## How Forking Works

This repo is designed to be fork-friendly:

### The `__SELF__` marker

The special `__SELF__` value in `secrets-config.yml` automatically resolves to
your profile repository (the repo named the same as your username).

**Example:**

- Original repo: `JacobPEvans/secrets-sync`, `__SELF__` →
  `JacobPEvans/JacobPEvans`
- Your fork: `yourname/secrets-sync`, `__SELF__` → `yourname/yourname`

This happens automatically - no manual find/replace needed.

### Dynamic owner resolution

The workflow uses `${{ github.repository_owner }}` to automatically detect your
GitHub username. When you fork:

**Original:**

```yaml
repositories:
  - my-app
```

Becomes: `JacobPEvans/my-app`

**Your fork:**

```yaml
repositories:
  - my-app
```

Becomes: `yourname/my-app`

**What you need to do:**

1. Fork the repo
1. Create your own PAT (see Setup)
1. Edit `secrets-config.yml` with your repo names
1. Set your secrets
1. Push

Everything else adapts automatically.

## Security

### Safety architecture (3 layers)

This repo is designed with defense-in-depth:

1. **Fine-grained PAT scope**
   - Token is scoped to a single owner (your username)
   - GitHub API blocks writes to repos you don't own
   - Only two permissions: Secrets (read/write), Metadata (read-only)

1. **Explicit allowlists**
   - `secrets-config.yml` contains explicit repository names
   - No regex patterns or wildcards
   - Every repo must be explicitly listed

1. **Branch protection (recommended)**
   - Enable branch protection on `main`
   - Require pull request reviews for `secrets-config.yml` changes
   - Use `CODEOWNERS` to require approval from specific users

### Branch protection setup

Protect the `main` branch to prevent unauthorized secret distribution:

1. Go to Settings → Branches → Add rule
1. Branch name pattern: `main`
1. Enable:
   - Require pull request reviews before merging
   - Require approvals: 1
   - Dismiss stale reviews when new commits are pushed
1. (Optional) Require status checks to pass (once you have CI)

### CODEOWNERS integration

Create `.github/CODEOWNERS` to require approval for critical files:

```text
# Require approval for security-sensitive files
/secrets-config.yml @your-username
/.github/workflows/sync-secrets.yml @your-username
```

Now changes to `secrets-config.yml` require your explicit approval.

### Secret rotation

To rotate the `GH_PAT_SECRETS_SYNC_ACTION` token:

1. Create a new PAT (see Setup → Creating the required PAT)
1. Update the secret:

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <your-username>/secrets-sync
```

1. Test with a dry run:

```bash
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=true
```

1. Revoke the old PAT at
   [github.com/settings/personal-access-tokens][pat-settings]

### What this does NOT do

- **No automatic secret rotation**: You must manually update secret values
- **No cross-org sync**: Token is scoped to one owner only
- **No secret deletion**: Removing a repo from config doesn't delete the secret
- **No value encryption**: GitHub handles encryption; this repo never sees
  plain values

## Limitations

Understanding what this tool does and doesn't do:

### What it does

✅ Syncs secret values from this repo to target repos

✅ Updates existing secrets automatically

✅ Creates new secrets in target repos

✅ Supports dry-run mode for testing

✅ Works with profile repositories via `__SELF__`

✅ Automatically adapts to forks (via `github.repository_owner`)

### What it doesn't do

❌ Rotate or generate secret values

❌ Delete secrets from target repos

❌ Sync across different GitHub organizations

❌ Retrieve secrets from target repos (one-way only)

❌ Validate secret formats or contents

❌ Sync repository settings (only secrets)

### Known constraints

- **PAT expiration**: Fine-grained PATs expire. Set a calendar reminder to
  rotate.
- **API rate limits**: Syncing many repos may hit rate limits (unlikely for
  <100 repos)
- **No rollback**: If you sync a bad secret value, you must manually fix it
- **Single owner**: Token works for one GitHub user/org, not across multiple

## Troubleshooting

### Workflow fails with "Resource not accessible by integration"

**Cause**: The `GH_PAT_SECRETS_SYNC_ACTION` token lacks permissions.

**Fix**:

1. Go to [github.com/settings/personal-access-tokens][pat-settings]
1. Find `secrets-sync-action` token
1. Verify permissions: `Secrets: Read and write`, `Metadata: Read-only`
1. Verify repository access includes target repos

### Workflow fails with "Not Found"

**Cause**: Target repository doesn't exist or token lacks access.

**Fix**:

1. Verify repo name in `secrets-config.yml` (check for typos)
1. Ensure repo owner matches token owner
1. Confirm repo exists: `gh repo view <owner>/<repo>`

### Secret not updating in target repo

**Cause**: Secret name mismatch or workflow didn't run.

**Fix**:

1. Check secret name spelling in `secrets-config.yml`
1. Verify secret exists in this repo:
   `gh secret list --repo <owner>/secrets-sync`
1. Check workflow logs: `gh run view --log`
1. Manually trigger: `gh workflow run sync-secrets.yml`

### `__SELF__` not resolving correctly

**Cause**: Workflow using cached owner name.

**Fix**:

1. Verify you forked (not cloned) the repo
1. Check workflow logs for `OWNER` and `PROFILE_REPO` values
1. Re-run workflow: `gh run rerun <run-id>`

### Dry run passes but production run fails

**Cause**: PAT permissions or rate limits.

**Fix**:

1. Check PAT hasn't expired:
   [github.com/settings/personal-access-tokens][pat-settings]
1. Review workflow logs for specific errors
1. Try syncing one secret at a time to isolate the issue

### Getting "Bad credentials" error

**Cause**: `GH_PAT_SECRETS_SYNC_ACTION` is missing, expired, or invalid.

**Fix**:

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <owner>/secrets-sync
```

Paste a fresh PAT when prompted.

## Contributing

Contributions welcome! This repo is designed for public use and improvements.

### Reporting issues

When reporting bugs, include:

- Workflow run URL or ID
- Relevant workflow logs (redact secret values!)
- `secrets-config.yml` structure (redact secret names if sensitive)
- Steps to reproduce

### Suggesting features

Open an issue with:

- Use case description
- Why current functionality doesn't address it
- Proposed solution

### Pull requests

1. Fork and create a branch
1. Make your changes
1. Test with dry-run mode
1. Update README if behavior changes
1. Submit PR with clear description

### Code of conduct

Be respectful and constructive. See GitHub's [Code of
Conduct](https://docs.github.com/en/site-policy/github-terms/github-community-code-of-conduct).

## License

See [LICENSE](LICENSE) file.

## Acknowledgments

Built with [jpoehnelt/secrets-sync-action][action] by
[@jpoehnelt](https://github.com/jpoehnelt).

[action]: https://github.com/jpoehnelt/secrets-sync-action
[pat]: https://github.com/settings/personal-access-tokens/new
[pat-settings]: https://github.com/settings/personal-access-tokens
