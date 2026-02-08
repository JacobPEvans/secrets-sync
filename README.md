# secrets-sync

[![Sync Secrets](https://github.com/JacobPEvans/secrets-sync/actions/workflows/sync-secrets.yml/badge.svg)](https://github.com/JacobPEvans/secrets-sync/actions/workflows/sync-secrets.yml)

Centralized GitHub Actions secret management. Define secrets once, sync them
everywhere automatically.

## What is this?

Manage GitHub Actions secrets across multiple repositories from a single
location. Instead of manually updating `API_TOKEN` in 20 different repos,
update it once here and let GitHub Actions sync it everywhere.

**Key features:**

- 📝 YAML configuration file (no workflow editing needed)
- 🔄 Matrix-based workflow (DRY principle)
- 🍴 Fork-friendly (auto-adapts to your username)
- 🔒 Secure by design (fine-grained PAT, explicit allowlists)
- 📚 Simple to use and maintain

## Quick Start

### 1. Fork this repository

The configuration automatically adapts to your GitHub username.

### 2. Create a Personal Access Token

1. Go to [github.com/settings/personal-access-tokens/new][pat]
2. Configure **exactly** as follows:
   - Token name: `secrets-sync-action`
   - Expiration: 90 days (recommended)
   - Repository access: Select the repositories that need secrets synced
   - Permissions:
     - **Secrets**: ✅ Read and write
     - **Metadata**: ✅ Read-only (auto-selected)
     - All other permissions: ❌ Unselected
3. Generate and **immediately copy** the token (you won't see it again)

### 3. Set the PAT

```bash
gh secret set GH_PAT_SECRETS_SYNC_ACTION --repo <your-username>/secrets-sync
```

### 4. Configure your secrets

Edit `secrets-config.yml`:

```yaml
secrets:
  - name: MY_API_TOKEN
    # Description of what this secret is for
    repositories:
      - my-app
      - my-other-app
      - __SELF__  # Your profile repo (<username>/<username>)
```

### 5. Add secret values

```bash
gh secret set MY_API_TOKEN --repo <your-username>/secrets-sync
```

### 6. Push and watch it sync

```bash
git add secrets-config.yml
git commit -m "Configure secret distribution"
git push
```

Done! Your secrets are now synced across all configured repos.

## Configuration

The `secrets-config.yml` file defines which secrets go to which repos:

```yaml
secrets:
  - name: SECRET_NAME
    # Comment: Purpose of this secret
    repositories:
      - repo-1        # Without owner prefix (added automatically)
      - repo-2
      - __SELF__      # Special marker for profile repo
```

**Key points:**

- Repository names are **without** owner prefix
- `__SELF__` resolves to your profile repository
- Comments are supported and encouraged
- Changes to this file trigger automatic syncs

See [Configuration Guide](docs/configuration.md) for detailed options.

## Usage

**Automatic**: Workflow runs when you push changes to `secrets-config.yml`

**Manual**:

```bash
# Dry run (preview only)
gh workflow run sync-secrets.yml --repo <user>/secrets-sync -f dry_run=true

# Production run
gh workflow run sync-secrets.yml --repo <user>/secrets-sync
```

## Documentation

- **[Setup Guide](docs/setup.md)** - Detailed PAT creation and configuration
- **[Configuration](docs/configuration.md)** - Advanced config options
- **[Security](SECURITY.md)** - Architecture and best practices
- **[Forking Guide](docs/forking.md)** - How fork-friendly features work
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions

## How it works

```text
secrets-config.yml → GitHub Actions → GitHub API → Target Repos
     (config)         (processes)     (syncs)      (updated)
```

1. You define mappings in `secrets-config.yml`
2. Workflow loads config and processes it with yq/jq
3. Matrix job syncs each secret to target repos
4. GitHub API writes secrets to repositories

The workflow uses `github.repository_owner` for fork-friendly operation and
supports the `__SELF__` marker for profile repositories.

## Security

- **Fine-grained PAT**: Scoped to your account only
- **Explicit allowlists**: Every repo must be listed
- **No wildcards**: No regex patterns or bulk operations
- **Branch protection**: Recommended for `secrets-config.yml` changes

See [Security Guide](SECURITY.md) for complete architecture.

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

See [LICENSE](LICENSE) file.

## Acknowledgments

Built with [jpoehnelt/secrets-sync-action][action] by
[@jpoehnelt](https://github.com/jpoehnelt).

[action]: https://github.com/jpoehnelt/secrets-sync-action
[pat]: https://github.com/settings/personal-access-tokens/new
