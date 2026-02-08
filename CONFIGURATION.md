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

## Adding a Secret

1. Edit `secrets-config.yml`
2. `gh secret set SECRET_NAME --repo <user>/secrets-sync`
3. Push

## Removing

Edit config and push. To fully remove:

```bash
gh secret remove SECRET_NAME --repo <user>/target-repo
```
