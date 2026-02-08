# Contributing

## Testing Changes

```bash
# Validate YAML
python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"

# Lint markdown
markdownlint-cli2 README.md docs/*.md

# Dry run
gh workflow run sync-secrets.yml -f dry_run=true
gh run view --log
```

## Commits

Use conventional commits: `type(scope): subject`

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## Code Style

- YAML: 2 spaces, sort lists alphabetically
- Markdown: Sequential numbering (1, 2, 3), markdownlint rules
- Shell: `set -euo pipefail`, handle errors

## Issues

Include workflow run URL, logs (redact secrets), steps to reproduce.
