# AI Agent Instructions for secrets-sync

## Debugging GitHub Actions Logs

When analyzing workflow failures in this repo:

**CRITICAL**: Do not assume the log line immediately after an
error message is the repo that caused the failure. The
`secrets-sync-action` logs errors asynchronously and continues
processing repos in parallel. An error may appear in the log
stream while the action is processing multiple repos
simultaneously.

To identify the actual failing repo:

1. Look for errors in the full log context (before and after)
2. Check which repos were successfully synced
3. Compare the success list against the target repo list
4. The missing repo is the one that failed

The action continues on error and logs all successes, so the
absence of a success message is more reliable than the presence
of an error message in a specific position.

## Repository Allowlist

This repo syncs secrets to an explicit allowlist only. The list
is maintained in the workflow file and represents repos that:

- Are actively maintained
- Require GitHub Actions secrets
- Have been explicitly approved for secret distribution

Do not add repos to the allowlist without explicit user approval.
