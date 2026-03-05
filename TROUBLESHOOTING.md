# Troubleshooting

## Common Issues

### "Resource not accessible by personal access token"

Three common causes, in order of likelihood:

1. **Repo not in the PAT's repository access list** — Fine-grained PATs grant
   access to a specific set of repositories. When you create a new repo, you
   must manually add it to the token's repository list in
   [GitHub Settings > Developer settings > Fine-grained tokens](https://github.com/settings/tokens?type=beta).
   Regenerating the token does NOT fix this — you must explicitly add the repo.

   The sync workflow validates PAT access at startup and fails fast with the
   exact repo names if any are inaccessible.

2. **PAT lacks permissions** — The token needs: `Secrets: Read & Write`,
   `Metadata: Read-only`. Codespaces secrets and Dependabot secrets permissions
   are also required if syncing to those scopes.

3. **Archived repository** — GitHub rejects secret writes to archived repos.
   Remove the repo from `secrets-config.yml`.

**How to diagnose which repo is failing:**

```bash
# Run a dry-run sync and check the logs
gh workflow run sync-secrets.yml --repo <user>/secrets-sync -f dry_run=true

# After it completes, check which repos are missing from "Set ... on" lines:
gh run view <run-id> --repo <user>/secrets-sync --log 2>&1 \
  | grep "Sync CLAUDE_CODE_OAUTH_TOKEN" \
  | grep -E "Set \`" \
  | sed 's/.*on //'
```

Compare against the repos in `secrets-config.yml` — any repo missing from
the output is inaccessible to the PAT.

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
