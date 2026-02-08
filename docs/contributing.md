# Contributing Guide

Thank you for considering contributing to secrets-sync!

## Ways to Contribute

- **Report bugs**: Open issues for bugs you encounter
- **Suggest features**: Propose new features or improvements
- **Improve docs**: Fix typos, clarify explanations, add examples
- **Submit PRs**: Fix bugs or implement new features

## Reporting Issues

When reporting bugs, include:

- Workflow run URL or ID
- Relevant workflow logs (redact secret values!)
- `secrets-config.yml` structure (redact secret names if sensitive)
- Steps to reproduce
- Expected vs actual behavior

## Suggesting Features

When proposing features, describe:

- **Use case**: What problem does this solve?
- **Current limitations**: Why doesn't existing functionality address it?
- **Proposed solution**: How would you implement it?
- **Alternatives considered**: What other approaches did you consider?

## Pull Request Process

1. **Fork and branch**: Create a branch from `main`
2. **Make changes**: Implement your changes
3. **Test thoroughly**:
   - Validate YAML syntax
   - Run markdown linting
   - Test with dry-run mode
4. **Update docs**: If behavior changes, update README or docs
5. **Commit with clear messages**: Use conventional commit format
6. **Submit PR**: Provide clear description and context

### Testing Your Changes

Before submitting:

```bash
# Validate YAML
python3 -c "import yaml; yaml.safe_load(open('secrets-config.yml'))"

# Lint markdown
markdownlint-cli2 README.md docs/*.md

# Test workflow (dry-run)
gh workflow run sync-secrets.yml -f dry_run=true

# Check logs
gh run view --log
```

### Commit Message Format

Use conventional commit format:

```text
type(scope): subject

body (optional)

footer (optional)
```

**Types:**

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**

```text
feat(config): add support for secret descriptions

Add optional description field to secret config entries
for better documentation.

Closes #123
```

```text
fix(workflow): prevent GITHUB_OUTPUT truncation

Use compact JSON output (jq -c) to prevent multiline
truncation in load-config job.
```

## Code Style

### YAML

- Use 2 spaces for indentation (no tabs)
- Always quote strings containing special characters
- Include comments for complex logic
- Sort lists alphabetically where appropriate

### Markdown

- Follow [markdownlint](https://github.com/DavidAnson/markdownlint) rules
- Use sequential numbering for ordered lists (1, 2, 3, not all 1s)
- Keep lines under 80 characters where practical
- Use fenced code blocks with language specifiers

### GitHub Actions Workflows

- Add comments explaining non-obvious logic
- Use `set -euo pipefail` for shell scripts
- Validate inputs and handle errors
- Keep jobs focused and single-purpose

## Code of Conduct

Be respectful and constructive. See GitHub's [Code of
Conduct](https://docs.github.com/en/site-policy/github-terms/github-community-code-of-conduct).

## Questions?

- Open an issue for questions
- Tag with `question` label
- Check existing issues first

Thank you for contributing!
