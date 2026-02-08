# Troubleshooting Guide

Common issues and solutions for secrets-sync.

## Workflow Failures

### "Resource not accessible by integration"

**Cause**: The `GH_PAT_SECRETS_SYNC_ACTION` token lacks permissions.

**Solution**:

1. Go to [github.com/settings/personal-access-tokens][pat-settings]
2. Find `secrets-sync-action` token
3. Verify permissions: `Secrets: Read and write`, `Metadata: Read-only`
4. Verify repository access includes target repos
5. If needed, regenerate token with correct permissions

### "Not Found" Error

**Cause**: Target repository doesn't exist or token lacks access.

**Solution**:

1. Verify repo name in `secrets-config.yml` (check for typos)
2. Ensure repo owner matches token owner
3. Confirm repo exists:

   ```bash
   gh repo view <owner>/<repo>
   ```

4. Check repo isn't private if token has limited access

### "Bad credentials" Error

**Cause**: `GH_PAT_SECRETS_SYNC_ACTION` is missing, expired, or invalid.

**Solution**:

```bash
# Create new token and set it
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <owner>/secrets-sync
```

Paste a fresh PAT when prompted.

## Secret Sync Issues

### Secret Not Updating in Target Repo

**Cause**: Secret name mismatch or workflow didn't run.

**Solution**:

1. Check secret name spelling in `secrets-config.yml`
2. Verify secret exists in this repo:

   ```bash
   gh secret list --repo <owner>/secrets-sync
   ```

3. Check workflow logs:

   ```bash
   gh run view --log
   ```

4. Manually trigger workflow:

   ```bash
   gh workflow run sync-secrets.yml --repo <owner>/secrets-sync
   ```

### Workflow Runs But No Secrets Synced

**Cause**: Dry-run mode enabled or matrix not generating correctly.

**Solution**:

1. Check if workflow was triggered with `dry_run=true`
2. Review `load-config` job output for matrix generation
3. Verify `secrets-config.yml` YAML syntax:

   ```bash
   python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"
   ```

## Configuration Issues

### `__SELF__` Not Resolving Correctly

**Cause**: Workflow using cached owner name or processing error.

**Solution**:

1. Verify you forked (not cloned) the repo
2. Check workflow logs for `OWNER` and `PROFILE_REPO` values:

   ```bash
   gh run view --log | grep -E "OWNER|PROFILE_REPO"
   ```

3. Re-run workflow:

   ```bash
   gh run rerun <run-id>
   ```

### YAML Syntax Errors

**Cause**: Invalid YAML in `secrets-config.yml`.

**Solution**:

```bash
# Validate syntax locally
python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"

# Check for common issues
# - Inconsistent indentation (use spaces, not tabs)
# - Missing colons after keys
# - Unquoted special characters
```

## Matrix Job Failures

### "Invalid JSON" Error in load-config

**Cause**: `yq` output is malformed or multiline.

**Solution**:

This should be fixed by the workflow using `jq -c` for compact output. If it
still occurs:

1. Check `load-config` job logs for the exact error
2. Verify `yq` and `jq` are available (they are pre-installed on GitHub runners)
3. Ensure no null values in `secrets-config.yml`

### Matrix Job Skipped

**Cause**: Matrix is empty or `load-config` failed.

**Solution**:

1. Check `load-config` job succeeded
2. Review its output for the generated matrix JSON
3. Verify `secrets-config.yml` contains at least one secret

## Performance Issues

### Workflow Takes Too Long

**Cause**: Syncing many secrets to many repos.

**Solution**:

- Normal: Matrix jobs run in parallel, should complete in 1-2 minutes
- If timeout occurs (>10 minutes), check for API rate limits
- Review workflow logs for stuck jobs

### API Rate Limits

**Cause**: Syncing many repos may hit GitHub API limits.

**Solution**:

1. Reduce frequency of sync runs
2. Use dry-run mode for testing
3. Wait for rate limit to reset (usually 1 hour)
4. Check rate limit status:

   ```bash
   gh api rate_limit
   ```

## Debugging Tips

### View Workflow Logs

```bash
# List recent runs
gh run list --repo <owner>/secrets-sync --workflow=sync-secrets.yml

# View specific run
gh run view <run-id> --log

# Watch live run
gh run watch
```

### Test Configuration Locally

```bash
# Validate YAML
python3 -c "import yaml; print(yaml.safe_load(open('secrets-config.yml')))"

# Simulate yq processing (requires yq installed)
yq eval -o=json '.secrets' secrets-config.yml | jq .
```

### Dry-Run Mode

Always test with dry-run first:

```bash
gh workflow run sync-secrets.yml \
  --repo <owner>/secrets-sync \
  -f dry_run=true
```

This previews changes without writing secrets.

## Getting Help

If issues persist:

1. Check [GitHub Actions status](https://www.githubstatus.com)
2. Review workflow file for recent changes
3. Open an issue with:
   - Workflow run URL
   - Relevant logs (redact secret values!)
   - `secrets-config.yml` structure (redact secret names if sensitive)
   - Steps to reproduce

[pat-settings]: https://github.com/settings/personal-access-tokens
