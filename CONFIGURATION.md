# Configuration

## Config File

```yaml
secrets:
  - name: SECRET_NAME
    repositories:
      - repo-1        # Without owner prefix
      - __SELF__      # Profile repo
```

## Key Points

- Repo names without owner prefix (added by workflow)
- `__SELF__` resolves to profile repo
- Sort repos alphabetically for maintainability
- Comments supported

## Two-Tier Distribution Model

Secrets fall into one of two tiers. Choose the tier based on distribution scope
and where the secret originates.

### Tier 1 — Broadly-shared secrets (this repo)

Use for secrets that go to many repos (4+) and originate in the secrets-sync
Doppler project.

Examples: Slack webhooks, Claude tokens, GitHub App keys, PATs.

**To add a Tier 1 secret:**

1. Edit `secrets-config.yml` — add the secret and its target repos
2. `gh secret set SECRET_NAME --repo <user>/secrets-sync`
3. Push — the sync workflow distributes it automatically

### Tier 2 — Infrastructure-specific secrets (per-repo runtime fetch)

Use for secrets that originate in `iac-conf-mgmt/prd` and are needed only by
specific infra repos. These secrets are **never added to `secrets-config.yml`**
and are never stored in GitHub Actions secrets.

Instead, infra repos hold `GH_ACTION_DOPPLER_IAC_CONF_MGMT` (a read-only service token for
`iac-conf-mgmt/prd`) and fetch secrets at CI runtime using
`dopplerhq/secrets-fetch-action`.

Examples: `MSSQL_SA_PASSWORD`, `QDRANT_API_KEY`.

**To add a new infra repo to Tier 2:**

1. Create a read-only Doppler service token for `iac-conf-mgmt/prd`
2. `gh secret set GH_ACTION_DOPPLER_IAC_CONF_MGMT --repo <user>/<infra-repo>`
   (or add the repo to the `_infra_repos` anchor in `secrets-config.yml` so
   secrets-sync distributes the token automatically)
3. Add a `dopplerhq/secrets-fetch-action` step to the repo's workflow

**Do NOT** add `iac-conf-mgmt/prd` secrets directly to `secrets-config.yml`.
This would copy values from their source, creating a second source of truth
that can drift.

### Decision table

| Signal | Tier |
| ------ | ---- |
| Secret goes to 4+ repos | Tier 1 |
| Secret originates in secrets-sync Doppler project | Tier 1 |
| Secret originates in `iac-conf-mgmt/prd` | Tier 2 |
| Only 1–3 infra repos need the secret | Tier 2 |

## Adding a Tier 1 Secret

1. Edit `secrets-config.yml`
2. `gh secret set SECRET_NAME --repo <user>/secrets-sync`
3. Push

## Removing

Edit config and push. To fully remove:

```bash
gh secret remove SECRET_NAME --repo <user>/target-repo
```
