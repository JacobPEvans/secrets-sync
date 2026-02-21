# Troubleshooting

## Common Issues

### "Resource not accessible"

Two common causes:

1. **PAT lacks permissions** — Check token scopes (`Secrets: Read/write`, `Metadata: Read-only`).
2. **Archived repository** — The GitHub API rejects secret writes to archived repos with this same
   error. Remove the repo from `secrets-config.yml`. Verify before adding:

   ```bash
   gh repo view <owner>/<repo> --json isArchived --jq '.isArchived'
   ```

### "Not Found"

Repo doesn't exist or typo in `secrets-config.yml`.

### "Bad credentials"

PAT expired or missing:

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <user>/secrets-sync
```

### Secret Not Updating

Check secret name spelling. Manually trigger:

```bash
gh workflow run sync-secrets.yml --repo <user>/secrets-sync
```

### Workflow Runs But No Sync

Check `load-config` job output. Validate YAML:

```bash
python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"
```

## Debugging

```bash
# View logs
gh run view --log

# Dry run
gh workflow run sync-secrets.yml -f dry_run=true
```
