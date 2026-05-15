# AGENTS.md — gjs-create-release

## Purpose
JavaScript GitHub Action (Node.js 12) that creates or updates GitHub Releases, uploads asset files via glob patterns, and optionally generates release notes. Wraps the upstream `softprops/action-gh-release` TypeScript package.

## Repository Map
```
gjs-create-release/
├── action.yml          # Action metadata — node12 runtime, main: dist/index.js
├── src/
│   ├── main.ts         # Entry point: parses inputs, orchestrates release creation
│   ├── github.ts       # GitHub API client wrappers (Octokit)
│   └── util.ts         # Glob expansion, file upload helpers
├── dist/
│   └── index.js        # Compiled + bundled output (committed, used at runtime)
├── __tests__/          # Jest test suite
├── package.json
└── README.md
```

## Tech Stack
- **Runtime**: Node.js 12 (`node12`)
- **Language**: TypeScript (compiled to `dist/index.js`)
- **GitHub API**: `@octokit/rest`
- **Bundler**: `@vercel/ncc` (bundles all deps into `dist/index.js`)
- **Test framework**: Jest

## Architecture Patterns
- `src/main.ts` is the entry point; reads all inputs via `@actions/core`, calls `src/github.ts` for API operations.
- `src/util.ts` handles glob expansion for the `files` input using the `glob` package.
- `dist/index.js` is the committed, bundled artefact that the runner executes directly — no `npm install` at runtime.
- Release creation is idempotent: if a release for the tag already exists, it is updated.

## Development Commands
```bash
# Install dependencies
npm install

# Build (TypeScript → dist/index.js)
npm run build

# Run tests
npm test

# Lint
npm run lint

# Full prepare (build + test)
npm run prepare
```

## Environment Setup
- Node.js 12+ (or 16+ for development tooling compatibility).
- `npm install` to restore `node_modules`.
- Set `GITHUB_TOKEN` environment variable for integration tests.

## Coding Conventions
- TypeScript strict mode; no implicit `any`.
- All GitHub API errors should be caught and re-thrown with context using `@actions/core.setFailed`.
- Keep `dist/index.js` committed and up-to-date — CI will fail if it is stale.
- Use `@actions/core.getInput` for all input reading; never access `process.env.INPUT_*` directly.

## Testing
```bash
npm test                        # Run Jest unit tests
npm test -- --coverage          # With coverage report
```
- Mock `@octokit/rest` for unit tests; use real API for integration tests.
- Test `fail_on_unmatched_files: true` with a glob that matches nothing.
- Test asset upload with files of varying sizes.

## Key Abstractions
- **`github.ts` `createRelease()`** — idempotent: creates or updates based on tag existence.
- **`util.ts` `globAssets()`** — expands newline-delimited glob patterns to file paths.
- **`append_body`** — when `true`, appends custom body to GitHub-generated notes rather than replacing.
- **`target_commitish`** — controls which commit the tag is created on; defaults to the default branch HEAD.

## Agentic Task Guidance
✅ Safe to add new inputs with sensible defaults — they are backwards compatible.
✅ Safe to upgrade `@octokit/rest` patch versions.
⚠️ After any TypeScript change, run `npm run build` and commit the updated `dist/index.js`.
⚠️ Node.js 12 is EOL; test on Node 16/20 if upgrading the runtime in `action.yml`.
❌ Do not modify `dist/index.js` directly — always edit `src/` and rebuild.
❌ Do not use `actions/upload-artifact` for release assets — use this action's `files` input.

## External Dependencies
- `@actions/core`, `@actions/github` (GitHub Actions toolkit)
- `@octokit/rest` (GitHub REST API client)
- `glob` (file pattern matching)
- Upstream: https://github.com/softprops/action-gh-release

## Common Pitfalls
- **Stale `dist/index.js`**: If `src/` is edited but `dist/` is not rebuilt, the action runs old code silently.
- **Node.js 12 EOL**: Actions platform still supports it but security patches have stopped; plan upgrade to node20.
- **Tag must exist**: The action does not create git tags; use `actions/checkout` + `git tag` before this step.
- **`files` glob paths**: Globs are resolved relative to `GITHUB_WORKSPACE`; use `**/*.tar.gz` not `./dist/**`.
- **Duplicate asset names**: Uploading an asset with the same name as an existing one fails; delete the old asset first or use a unique naming scheme.
