# Security

## Branch Protection Required

Enable branch protection on `main` before production use. Without it, anyone
with write access can modify config and trigger syncs.

## Architecture

### Two-Tier Secret Distribution

Secrets are distributed using one of two patterns depending on how broadly they
are shared and where they originate:

#### Tier 1 — Broadly-shared secrets (secrets-sync workflow)

Secrets distributed to many repos (Slack webhooks, Claude tokens, GitHub App
keys, PATs). The Doppler auto-sync pushes these into secrets-sync's GitHub
Actions secrets, and the sync workflow distributes them to target repos.

- Low sensitivity or infrequently rotated
- Distributed to 4–20 repos each
- Managed via `secrets-config.yml`

#### Tier 2 — Infrastructure-specific secrets (per-repo Doppler runtime fetch)

Secrets from `iac-conf-mgmt/prd` needed only by specific infra repos (e.g.
`MSSQL_SA_PASSWORD`, `QDRANT_API_KEY`). Infra repos fetch these at CI runtime
using `dopplerhq/secrets-fetch-action` — they are never stored in GitHub
Actions secrets.

- Scoped to one Doppler config (`iac-conf-mgmt/prd`)
- Never duplicated into the secrets-sync Doppler project
- Each infra repo holds `GH_ACTION_DOPPLER_IAC_CONF_MGMT` (distributed via secrets-sync Tier 1)
- The token itself is read-only and grants access to the full `iac-conf-mgmt/prd` config

```text
secrets-sync repo
  └─ distributes GH_ACTION_DOPPLER_IAC_CONF_MGMT ──► ansible-proxmox-apps
                                            │
                                            └─ CI runtime: dopplerhq/secrets-fetch-action
                                                 └─ fetches from iac-conf-mgmt/prd
                                                      ├─ MSSQL_SA_PASSWORD
                                                      └─ QDRANT_API_KEY
```

**Why this split?** Adding `iac-conf-mgmt/prd` to the Tier 1 auto-sync would
expose every secret in that Doppler project just to get 2 values. Runtime fetch
keeps the blast radius contained: secrets-sync never sees the infra secrets,
and infra secrets are fetched only during CI runs.

### Tier 1 Secret Storage

1. **PAT scope**: Token scoped to owner, `Secrets: Read/write`, `Metadata: Read-only`
2. **Explicit allowlists**: Every repo must be listed in `secrets-config.yml`
3. **Branch protection**: Require PR reviews for config changes

### Tier 2 Token Scope

The `GH_ACTION_DOPPLER_IAC_CONF_MGMT` service token is:

- **Read-only** — cannot modify Doppler secrets
- **Scoped to `iac-conf-mgmt/prd`** — cannot access other Doppler projects
- **Stored as a GitHub Actions secret** — encrypted at rest, masked in logs
- Protected by branch protection + CODEOWNERS against unauthorized workflow edits

Remaining risk: on the Doppler free plan, service tokens scope to the entire
config (not individual secrets). Mitigation: the token is read-only, and
access to it is limited to repos that genuinely need infra secrets.

## Setup Branch Protection

Settings → Branches → Add rule:

- Pattern: `main`
- Require PR reviews
- Require approvals: 1

## CODEOWNERS

```text
/secrets-config.yml @your-username
/.github/workflows/sync-secrets.yml @your-username
```

## Limitations

Does NOT protect against:

- Compromised GitHub account
- Malicious collaborators with write access
- Workflow log exposure
- Secret values in git history
