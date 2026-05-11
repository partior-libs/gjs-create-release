# AGENTS.md — gjs-create-release

## Purpose

GitHub JavaScript Action (Node.js 12, TypeScript source compiled to `dist/index.js`) that creates or updates a GitHub Release for a given tag. Supports uploading binary assets via glob patterns, auto-generating release notes from commit history, draft/pre-release flags, and appending to existing release bodies.

## Repository Map

| Path | Role |
|------|------|
| `action.yml` | JavaScript action definition — `using: node12`, `main: dist/index.js` |
| `src/main.ts` | Entry point — parses inputs, orchestrates release creation and asset upload |
| `src/github.ts` | GitHub API client — Octokit with retry and throttling plugins; release CRUD + asset upload |
| `src/util.ts` | Utilities — file glob resolution, asset MIME type detection |
| `dist/index.js` | Compiled and minified bundle (committed to repo — required for action execution) |
| `__tests__/github.test.ts` | Unit tests for `github.ts` |
| `__tests__/util.test.ts` | Unit tests for `util.ts` |
| `tests/data/foo/bar.txt` | Test fixture file |
| `package.json` | npm dependencies and build/test/format scripts |
| `tsconfig.json` | TypeScript compiler configuration |
| `jest.config.js` | Jest test runner configuration |
| `release.sh` | Helper script for publishing a new version of this action |
| `.github/workflows/main.yml` | CI workflow — build, test, release |

## Tech Stack

| Component | Version / Detail |
|-----------|-----------------|
| Action type | GitHub JavaScript Action |
| Runtime | Node.js 12 |
| Language | TypeScript 3.7.x |
| Bundler | `@zeit/ncc` (minified single-file bundle) |
| GitHub API client | `@actions/github` 5.x + `@octokit/plugin-retry` 3.x + `@octokit/plugin-throttling` 3.x |
| Test framework | Jest 24.x + `ts-jest` |
| Linter/formatter | Prettier 1.19.1 |

## Architecture Patterns

- **JavaScript action with compiled bundle** — TypeScript source in `src/` is compiled by `ncc` into `dist/index.js`. The compiled bundle is committed and is what GitHub Actions executes; the TypeScript source is for development.
- **Retry + throttle plugins** — the Octokit client wraps the GitHub API with automatic retry on transient errors and rate-limit throttling.
- **Glob-based asset upload** — `util.ts` resolves `files` input glob patterns against the workspace and uploads each matched file using the release `upload_url`.

## Development Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install all dependencies |
| `npm run build` | Compile TypeScript → `dist/index.js` via `ncc` |
| `npm test` | Run Jest unit tests |
| `npm run fmt` | Format source files with Prettier |
| `npm run fmtcheck` | Check formatting without modifying files |
| `bash release.sh` | Publish a new version (tags and pushes) |

## Environment Setup

| Variable/Secret | Description | Example |
|----------------|-------------|---------|
| `GITHUB_TOKEN` | Required for GitHub API calls; set automatically by Actions | `${{ github.token }}` |
| `token` input | Override token (e.g., PAT for cross-repo releases) | `${{ secrets.GH_PAT }}` |
| `NODE_VERSION` | Node.js 12 required at build time | — |

## Key Abstractions

- `src/main.ts` — reads all action inputs via `@actions/core.getInput()`, calls `github.ts` functions, and sets outputs.
- `github.ts: GitHub` class — wraps Octokit; provides `getRelease()`, `createRelease()`, `updateRelease()`, `uploadReleaseAsset()` methods.
- `util.ts: paths()` — resolves the `files` input globs to actual file paths on the runner filesystem.
- `dist/index.js` — the executable; **must be regenerated and committed after any source change** via `npm run build`.

## Agentic Task Guidance

- ✅ Safe: Reading source files; running `npm test`; running `npm run fmtcheck`; updating input descriptions in `action.yml`.
- ⚠️ Review: Modifying `src/github.ts` API calls (risk of breaking release creation or asset upload); upgrading Octokit dependencies (check for breaking API changes); changing the Node.js version in `action.yml`.
- ❌ Avoid: Modifying `dist/index.js` directly (always regenerate via `npm run build`); committing source changes without rebuilding `dist/`; using `node12` runtime for new code (deprecated — plan migration to `node20`).

## External Dependencies & Integrations

- **GitHub Releases API** — creates/updates releases, uploads assets.
- **GitHub Discussions API** — links release to a discussion category if `discussion_category_name` is set.
- **npm registry** — `@actions/core`, `@actions/github`, Octokit plugins.

## Common Pitfalls

- **`dist/index.js` must be committed** — the action runs from the compiled bundle, not the TypeScript source. Forgetting to rebuild and commit `dist/` after source changes is the most common mistake.
- Node.js 12 is deprecated by GitHub Actions — plan migration to `node20` and update `action.yml`'s `using` field.
- Asset MIME types are detected by file extension via the `mime` package — unusual extensions may default to `application/octet-stream`.
- The `files` glob is resolved relative to the `GITHUB_WORKSPACE` directory — ensure build artifacts are created before this action runs.
- `fail_on_unmatched_files: true` is useful in CI to catch misconfigured globs early, but may cause unexpected failures if optional files are missing.
