# secrets-sync

Centralized GitHub Actions secret management using
[jpoehnelt/secrets-sync-action][action].

Secrets are defined once in this repo and distributed
to target repos automatically.

## Secret Distribution Map

| Secret                      | Target Repos          | Method        |
| --------------------------- | --------------------- | ------------- |
| `GH_PAT_WORKFLOW_DISPATCH`  | `ai-assistant-*` etc. | Explicit list |
| `CLAUDE_CODE_OAUTH_TOKEN`   | All `JacobPEvans/`    | Regex         |
| `GH_PAC_METRICS_TOKEN`      | `JacobPEvans` profile | Explicit list |

Explicit targets for `GH_PAT_WORKFLOW_DISPATCH`:
`ai-assistant-instructions`, `claude-code-plugins`, `.github`.

## Safety Architecture (3 Layers)

1. **Fine-grained PAT** (`GH_PAT_SECRETS_SYNC_ACTION`): Scoped to
   `JacobPEvans` owner only, with `Secrets: Read and write`
   and `Metadata: Read-only` permissions. The GitHub API
   blocks access to any repo outside this owner.
1. **Repository patterns**: Explicit repo lists for targeted
   syncs; anchored regex (`^JacobPEvans/`) for "all repos".
   No unanchored patterns.
1. **Private repo**: Workflow config is invisible without
   explicit access.

## Triggers

- **Push to main**: Runs when the workflow file changes.
- **Manual dispatch**: With optional dry-run mode.

## Adding a New Secret

1. Add a new job to `.github/workflows/sync-secrets.yml`:

```yaml
  sync-new-secret:
    name: "Sync NEW_SECRET_NAME"
    runs-on: ubuntu-latest
    steps:
      - uses: jpoehnelt/secrets-sync-action@<sha>
        with:
          SECRETS: |
            ^NEW_SECRET_NAME$
          REPOSITORIES: |
            JacobPEvans/target-repo
          REPOSITORIES_LIST_REGEX: false
          DRY_RUN: ${{ inputs.dry_run || 'false' }}
          GITHUB_TOKEN: ${{ secrets.GH_PAT_SECRETS_SYNC_ACTION }}
        env:
          NEW_SECRET_NAME: ${{ secrets.NEW_SECRET_NAME }}
```

1. Set the secret value:
   `gh secret set NEW_SECRET_NAME --repo JacobPEvans/secrets-sync`
1. Push the workflow change (triggers a sync automatically).

## PAT Rotation

The `GH_PAT_SECRETS_SYNC_ACTION` fine-grained PAT powers all sync
jobs. To rotate:

1. Create a new fine-grained PAT at
   [github.com/settings/personal-access-tokens/new][pat]:
   - **Name**: `secrets-sync-action`
   - **Resource owner**: `JacobPEvans`
   - **Repository access**: All repositories
   - **Permissions**: Secrets (Read and write),
     Metadata (Read-only)
1. Update the secret:
   `gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo JacobPEvans/secrets-sync`
1. Trigger a dry run to verify: dispatch workflow with
   `dry_run: true`.

## Verification

```bash
# Trigger dry run
gh workflow run sync-secrets.yml \
  --repo JacobPEvans/secrets-sync -f dry_run=true

# Check workflow logs
gh run list --repo JacobPEvans/secrets-sync \
  --workflow=sync-secrets.yml

# Spot-check a target repo
gh secret list \
  --repo JacobPEvans/ai-assistant-instructions
```

[action]: https://github.com/jpoehnelt/secrets-sync-action
[pat]: https://github.com/settings/personal-access-tokens/new
