# Forking Guide

How fork-friendly features work in secrets-sync.

## Overview

This repo is designed to be easily forked and adapted to your account without
manual find/replace of usernames or repository names.

## Dynamic Owner Resolution

The workflow uses `${{ github.repository_owner }}` to automatically detect your
GitHub username at runtime.

**Example:**

Original workflow:

```yaml
OWNER="${{ github.repository_owner }}"
```

When you fork:

- Original: `OWNER` = `JacobPEvans`
- Your fork: `OWNER` = `yourname`

This happens automatically - no changes needed.

## The `__SELF__` Marker

The special `__SELF__` value in `secrets-config.yml` resolves to your profile
repository (the repo named the same as your username).

**Example:**

```yaml
repositories:
  - __SELF__  # Profile repository
```

Resolution:

- Original: `JacobPEvans/secrets-sync`, `__SELF__` → `JacobPEvans/JacobPEvans`
- Your fork: `yourname/secrets-sync`, `__SELF__` → `yourname/yourname`

This is processed at runtime in the `load-config` job:

```bash
if . == "__SELF__" then env(PROFILE_REPO)
else env(OWNER) + "/" + . end
```

## How It Works

### 1. Fork the Repository

Click "Fork" on GitHub. The repo copies to your account.

### 2. Repository Names Adapt

In `secrets-config.yml`:

```yaml
repositories:
  - my-app
```

Original: `JacobPEvans/my-app`
Your fork: `yourname/my-app`

The owner prefix is added dynamically during workflow execution.

### 3. Workflow Processing

The `load-config` job:

1. Reads `secrets-config.yml`
2. Gets `OWNER` from `github.repository_owner`
3. Replaces `__SELF__` with `OWNER` (your username)
4. Adds `OWNER/` prefix to all repo names
5. Outputs formatted JSON for the matrix

## What You Need to Do

When you fork, only these steps are required:

1. **Create your own PAT** (see [Setup Guide](setup.md))
2. **Edit `secrets-config.yml`** with your repository names
3. **Set your secrets** using `gh secret set`
4. **Push changes**

Everything else adapts automatically:

- Owner name resolution
- Profile repository (`__SELF__`)
- Workflow execution context

## What Does NOT Need Changing

You do **not** need to:

- Search/replace usernames in the workflow
- Update any `github.repository_owner` references
- Modify the `load-config` job logic
- Change action versions or SHAs

## Verification

After forking, verify the auto-adaptation:

```bash
# Trigger a dry-run
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=true

# Check the logs
gh run view --log
```

Look for:

- `OWNER` variable shows your username
- `PROFILE_REPO` resolves to your profile repo
- Repository names have your username prefix

## Common Issues

### `__SELF__` Not Resolving

If `__SELF__` doesn't resolve correctly:

1. Verify you forked (not cloned) the repo
2. Check workflow logs for `OWNER` and `PROFILE_REPO` values
3. Re-run the workflow

### Owner Prefix Missing

If repos are synced without owner prefix:

1. Check `secrets-config.yml` doesn't already include owner prefix
2. Verify workflow is using `github.repository_owner`
3. Review `load-config` job output

## Benefits of Fork-Friendly Design

- **No manual editing**: Fork and configure, no search/replace needed
- **Portable**: Works the same way for all users
- **Maintainable**: Updates to the workflow don't break forks
- **DRY principle**: Owner name defined once, used everywhere
- **Error-resistant**: Less manual editing = fewer mistakes

## Next Steps

- [Configure your secrets](configuration.md)
- [Review security best practices](security.md)
