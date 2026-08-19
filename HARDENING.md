<!-- markdownlint-disable -->

# Hardening Report: googleapis--release-please-action/v4.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **googleapis--release-please-action/v4.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use tag- or branch-based `uses:` references instead of full 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

.github/workflows/ci.yaml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4 (used in 3 jobs)

.github/workflows/release-please.yaml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - uses: googleapis/code-suggester@v4
  - uses: googleapis/release-please-action@main (used in 2 jobs — @main is a mutable branch ref)

Locations:

- `.github/workflows/ci.yaml:14`
- `.github/workflows/ci.yaml:15`
- `.github/workflows/ci.yaml:27`
- `.github/workflows/ci.yaml:28`
- `.github/workflows/ci.yaml:38`
- `.github/workflows/ci.yaml:39`
- `.github/workflows/release-please.yaml:14`
- `.github/workflows/release-please.yaml:15`
- `.github/workflows/release-please.yaml:25`
- `.github/workflows/release-please.yaml:40`
- `.github/workflows/release-please.yaml:43`
- `.github/workflows/release-please.yaml:63`

### script-injection (severity: high)

The 'tag major and minor versions' run: block in release-please.yaml directly interpolates GitHub Actions expressions inside shell commands (sub-rule a). This includes `${{ secrets.GITHUB_TOKEN }}` embedded in a git remote URL, and `${{ steps.release.outputs.major }}` / `${{ steps.release.outputs.minor }}` used as git tag arguments. Any of these values are substituted into the shell command string before the shell parses it, allowing shell metacharacters to be injected. Specifically:
  - `git remote add gh-token "https://${{ secrets.GITHUB_TOKEN}}@github.com/..."` — token embedded directly in shell string
  - `git tag -d v${{ steps.release.outputs.major }} || true` — step output directly in shell
  - `git tag -d v${{ steps.release.outputs.major }}.${{ steps.release.outputs.minor }} || true` — step outputs directly in shell
  - (and several more similar lines)
All `${{ ... }}` expressions inside run: blocks are script-injection risks regardless of context.

Locations:

- `.github/workflows/release-please.yaml:51`
- `.github/workflows/release-please.yaml:54`
- `.github/workflows/release-please.yaml:55`
- `.github/workflows/release-please.yaml:56`
- `.github/workflows/release-please.yaml:57`
- `.github/workflows/release-please.yaml:58`
- `.github/workflows/release-please.yaml:59`
- `.github/workflows/release-please.yaml:60`
- `.github/workflows/release-please.yaml:61`

### missing-permissions (severity: medium)

The workflow file ci.yaml has no top-level `permissions:` key and none of its jobs (test, windows, build-dist) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents and other scopes). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/ci.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all action references in ci.yaml and release-please.yaml to full 40-char commit SHAs (actions/checkout@11d5960a, actions/setup-node@49933ea5, googleapis/code-suggester@589b3ac1, googleapis/release-please-action@0b6b3fc0), preserving original tags as comments.
2. script-injection: In release-please.yaml 'tag major and minor versions' step, moved ${{ secrets.GITHUB_TOKEN }}, ${{ steps.release.outputs.major }}, and ${{ steps.release.outputs.minor }} into an env: block as GITHUB_TOKEN, MAJOR, and MINOR respectively. Shell script now references plain env vars.
3. missing-permissions: Added top-level 'permissions: contents: read' to ci.yaml (the workflow only reads code and runs tests, so read-only is sufficient).

