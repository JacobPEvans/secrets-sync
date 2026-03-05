# Changelog

## [0.2.0](https://github.com/JacobPEvans/secrets-sync/compare/v0.1.0...v0.2.0) (2026-03-05)


### Features

* add ANTHROPIC_API_KEY to all_repos distribution ([7812d62](https://github.com/JacobPEvans/secrets-sync/commit/7812d621f2f8a98845d2474e263638ac02f30a1c))
* add GH_SLACK_CHANNEL_ID_GITHUB_AUTOMATION to all repos ([a5b4850](https://github.com/JacobPEvans/secrets-sync/commit/a5b4850df21fbd565474c2d601add5fc6fc86e48))
* add GH_SLACK_WEBHOOK_URL_GITHUB_AUTOMATION to all repos ([e102e39](https://github.com/JacobPEvans/secrets-sync/commit/e102e39775f44a4d5b02ca05d1ac0a9bfbf8d2ef))
* enable auto-merge on release-please PRs ([032bfd0](https://github.com/JacobPEvans/secrets-sync/commit/032bfd017cff6b6eb66ab145173ad4329a5a40f3))
* **gh-aw:** add malicious-code-scan workflow, update secrets config ([ce0d946](https://github.com/JacobPEvans/secrets-sync/commit/ce0d9460b50c131d8c97d45c84ade5b4ebfbf7c3))
* secrets-sync - centralized GitHub Actions secret management ([5c067fb](https://github.com/JacobPEvans/secrets-sync/commit/5c067fb03a0d20515e3717eb882eb168c5823159))
* yaml anchors, new secrets, badges, security doc fix ([46e3ee3](https://github.com/JacobPEvans/secrets-sync/commit/46e3ee3552ed07f888cb550d31a8b9cf208a5641))


### Bug Fixes

* add --repo flag to gh pr merge for non-checkout context ([b08c871](https://github.com/JacobPEvans/secrets-sync/commit/b08c8717d1aae2bcdc91f769e5687e032cac4d3d))
* address PR review feedback on gh-aw workflows ([cad0a76](https://github.com/JacobPEvans/secrets-sync/commit/cad0a767d05fee5c6489792400aeb922310319fc))
* comment out ANTHROPIC_API_KEY sync for now ([d1ff953](https://github.com/JacobPEvans/secrets-sync/commit/d1ff953125342aed347392a9173ae87f5ca90dad))
* parse PR number from release-please JSON output ([6822504](https://github.com/JacobPEvans/secrets-sync/commit/682250429c91dfffbf9efeab415387ffd1b2e6ca))
* remove ANTHROPIC_API_KEY alias, fix alphabetical ordering ([debb48f](https://github.com/JacobPEvans/secrets-sync/commit/debb48f30aa3ebadb78f73a0c0b2f6b96d965563))
* use --rebase merge method for release-please auto-merge ([dc61f9a](https://github.com/JacobPEvans/secrets-sync/commit/dc61f9a16bc5b518d91ecd2f49ad47b642c605ec))
