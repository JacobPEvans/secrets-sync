# Configuration Guide

Detailed guide to configuring `secrets-config.yml`.

## File Structure

```yaml
secrets:
  - name: SECRET_NAME
    # Comment: Purpose of this secret
    repositories:
      - repo-1        # Without owner prefix
      - repo-2
      - __SELF__      # Special marker for profile repo
```

## Key Concepts

### Repository Names

Repository names are specified **without** the owner prefix:

- ✅ **Correct**: `my-app`
- ❌ **Wrong**: `username/my-app`

The workflow automatically adds your username (from `github.repository_owner`)
at runtime.

### The `__SELF__` Marker

The special `__SELF__` value resolves to your profile repository (the repo
named the same as your username):

- Original: `JacobPEvans/secrets-sync`, `__SELF__` → `JacobPEvans/JacobPEvans`
- Your fork: `yourname/secrets-sync`, `__SELF__` → `yourname/yourname`

This happens automatically - no manual find/replace needed when forking.

### Comments

Comments are supported and encouraged:

```yaml
secrets:
  - name: DEPLOY_TOKEN
    # OAuth token for deployment automation
    repositories:
      - prod-app      # Production deployment
      - staging-app   # Staging environment
```

Comments help document:

- What each secret is for
- Why specific repos need it
- Any special considerations

## Adding a Secret

1. Edit `secrets-config.yml` and add a new entry:

   ```yaml
   - name: NEW_SECRET_NAME
     # Purpose of this secret
     repositories:
       - target-repo-1
       - target-repo-2
   ```

2. Set the secret value in this repo:

   ```bash
   gh secret set NEW_SECRET_NAME --repo <your-username>/secrets-sync
   ```

3. Push the config change:

   ```bash
   git add secrets-config.yml
   git commit -m "Add NEW_SECRET_NAME to sync"
   git push
   ```

The workflow runs automatically and syncs the new secret.

## Removing a Repository

Edit `secrets-config.yml` and remove the repo from the list. The workflow does
**not** delete secrets from repos (by design - deletion would be destructive).

To fully remove:

1. Edit `secrets-config.yml` and push
2. Manually delete from the target repo:

   ```bash
   gh secret remove SECRET_NAME --repo <your-username>/target-repo
   ```

## Best Practices

### Alphabetical Sorting

Sort repository lists alphabetically for maintainability:

```yaml
repositories:
  - app-backend
  - app-frontend
  - app-mobile
  - infrastructure
```

This makes it easy to:

- Check if a repo is already in the list
- Spot duplicates
- Find specific repos quickly

### Grouping Related Repos

Use comments to group related repositories:

```yaml
repositories:
  # Production services
  - prod-api
  - prod-web
  - prod-worker

  # Staging services
  - staging-api
  - staging-web
```

### Document Secret Purposes

Always include a comment explaining what each secret is for:

```yaml
- name: SENTRY_DSN
  # Error tracking and monitoring (Sentry.io)
  repositories:
    - backend-api
    - frontend-app
```

## Example Configurations

### Simple Setup

```yaml
secrets:
  - name: DEPLOY_KEY
    # SSH key for deployment
    repositories:
      - my-app
```

### Multi-Environment

```yaml
secrets:
  - name: DATABASE_URL
    # PostgreSQL connection string
    repositories:
      - api-prod
      - api-staging
      - api-dev
```

### Profile Repository Secrets

```yaml
secrets:
  - name: METRICS_TOKEN
    # GitHub profile metrics token
    repositories:
      - __SELF__  # Profile repo only
```

## Validation

Before pushing config changes:

```bash
# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"

# Trigger dry-run
gh workflow run sync-secrets.yml \
  --repo <your-username>/secrets-sync \
  -f dry_run=true
```

## Next Steps

- [Learn about security best practices](security.md)
- [Troubleshoot common issues](troubleshooting.md)
