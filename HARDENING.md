<!-- markdownlint-disable -->

# Hardening Report: googleapis--release-please-action/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **googleapis--release-please-action/v5.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or branch refs instead of pinned 40-character SHA commits, making the workflows vulnerable to supply-chain attacks if the referenced action is compromised or the tag is moved.

In `.github/workflows/ci.yaml`:
- `actions/checkout@v4` (appears in all 3 jobs)
- `actions/setup-node@v4` (appears in all 3 jobs)

In `.github/workflows/release-please.yaml`:
- `actions/checkout@v4`
- `actions/setup-node@v4`
- `googleapis/code-suggester@v4`
- `googleapis/release-please-action@main` (appears twice — one in `release-please-release` job and one in `release-please-pr` job; `@main` is a branch ref and especially dangerous)

Locations:

- `.github/workflows/ci.yaml:13`
- `.github/workflows/ci.yaml:14`
- `.github/workflows/release-please.yaml:13`
- `.github/workflows/release-please.yaml:14`
- `.github/workflows/release-please.yaml:22`
- `.github/workflows/release-please.yaml:37`
- `.github/workflows/release-please.yaml:45`
- `.github/workflows/release-please.yaml:68`

### missing-permissions (severity: medium)

`.github/workflows/ci.yaml` has no top-level `permissions:` key and none of its three jobs (`test`, `windows`, `build-dist`) define a job-level `permissions:` block. This means the workflow runs with the default repository permissions, which may be broader than necessary (e.g. write access to contents and packages on some repository configurations). A minimal `permissions: read-all` or specific scopes should be declared.

Locations:

- `.github/workflows/ci.yaml:1`

### script-injection (severity: high)

The `run:` block in the `release-please-release` job of `.github/workflows/release-please.yaml` (step: "tag major and minor versions") directly interpolates GitHub Actions expressions inside shell commands, violating sub-rule (a). The following expressions are interpolated:

- `${{ secrets.GITHUB_TOKEN }}` — embedded directly in a git remote URL: `git remote add gh-token "https://${{ secrets.GITHUB_TOKEN}}@github.com/..."`. Even though this is a GitHub-controlled secret, any `${{ ... }}` inside a `run:` block is a script-injection finding per the check rules.
- `${{ steps.release.outputs.major }}` — used repeatedly in `git tag` and `git push` commands. `steps.*.outputs.*` is a workflow-controllable context; if the release-please action step is compromised or its output is manipulated, this allows arbitrary shell command injection.
- `${{ steps.release.outputs.minor }}` — same issue as above.

All these values should be moved to `env:` variables and referenced as quoted shell variables (e.g. `"$MAJOR"`) instead of being interpolated directly into the shell script.

Locations:

- `.github/workflows/release-please.yaml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings. 1) unpinned-uses: Pinned all action references to full 40-char SHAs in both workflow files with original tags preserved as comments. 2) missing-permissions: Added top-level permissions block with contents:read to ci.yaml. 3) script-injection: Moved all ${{}} expressions in the tag major and minor versions step into an env: block, referencing them as quoted shell variables.

