# gjs-create-release

> GitHub JavaScript Action (Node.js/TypeScript) for creating and publishing GitHub Releases, with support for asset uploads, auto-generated release notes, and draft/pre-release flags.

## Overview

`gjs-create-release` is a Node.js action (built from TypeScript, compiled to `dist/index.js`) based on [softprops/action-gh-release](https://github.com/softprops/action-gh-release). It calls the GitHub Releases API to create or update a release for a given tag, optionally uploading binary assets, generating release notes automatically, and linking to discussions.

## Usage

```yaml
- uses: partior-libs/gjs-create-release@main
  with:
    tag_name: ${{ github.ref_name }}
    name: "Release ${{ github.ref_name }}"
    body: "Automated release"
    files: |
      dist/*.tar.gz
      dist/*.zip
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `body` | Release notes body text | No | — |
| `body_path` | Path to a file containing the release notes body | No | — |
| `name` | Custom release name (defaults to tag name) | No | — |
| `tag_name` | Git tag for the release (defaults to `GITHUB_REF`) | No | — |
| `draft` | Create as a draft release | No | `false` |
| `prerelease` | Mark as a pre-release | No | `false` |
| `files` | Newline-delimited glob patterns of asset files to upload | No | — |
| `fail_on_unmatched_files` | Fail if any `files` glob matches nothing | No | `false` |
| `repository` | Target repository in `owner/repo` format | No | *(current repo)* |
| `token` | GitHub token for authentication | No | `github.token` |
| `target_commitish` | Branch or commit SHA the tag points to | No | — |
| `discussion_category_name` | Link release to a discussion in this category | No | — |
| `generate_release_notes` | Auto-generate release name and body from commits | No | `false` |
| `append_body` | Append to existing body instead of overwriting | No | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `url` | URL to the Release HTML page |
| `id` | Numeric Release ID |
| `upload_url` | URL for uploading additional assets to the release |
| `assets` | JSON array of uploaded asset metadata |

## Prerequisites

- The `token` must have `contents:write` permission (the default `github.token` has this on `push` events).
- Asset files referenced in `files` must exist on the runner before this step runs.

## Examples

### Simple release on tag push

```yaml
on:
  push:
    tags: ["v*"]

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: partior-libs/gjs-create-release@main
        with:
          tag_name: ${{ github.ref_name }}
          generate_release_notes: "true"
```

### Release with binary assets

```yaml
- name: Build artifacts
  run: make dist

- uses: partior-libs/gjs-create-release@main
  with:
    tag_name: ${{ github.ref_name }}
    name: "Release ${{ github.ref_name }}"
    body_path: CHANGELOG.md
    files: |
      dist/myapp-linux-amd64
      dist/myapp-darwin-arm64
      dist/*.tar.gz
```

### Draft pre-release

```yaml
- uses: partior-libs/gjs-create-release@main
  with:
    tag_name: ${{ github.ref_name }}-rc
    draft: "true"
    prerelease: "true"
    generate_release_notes: "true"
```

## Project Structure

```
gjs-create-release/
├── action.yml                  # JavaScript action definition (runs via node12 + dist/index.js)
├── src/
│   ├── main.ts                 # Entry point — parses inputs, calls GitHub API
│   ├── github.ts               # GitHub API client wrapper (Octokit + retry/throttle plugins)
│   └── util.ts                 # File globbing, asset upload helpers
├── __tests__/
│   ├── github.test.ts          # Unit tests for GitHub client
│   └── util.test.ts            # Unit tests for utilities
├── tests/
│   └── data/foo/bar.txt        # Test fixture data
├── package.json                # npm dependencies and build scripts
├── tsconfig.json               # TypeScript compiler configuration
├── jest.config.js              # Jest test runner configuration
├── release.sh                  # Helper script for publishing new versions
└── .github/workflows/
    └── main.yml                # CI/CD workflow — build, test, release
```

## Development

```bash
# Install dependencies
npm install

# Build (TypeScript → dist/index.js)
npm run build

# Run tests
npm test

# Format code
npm run fmt
```

## Contributing

1. Fork the repository and create a feature branch.
2. Make changes in `src/`, run `npm run build` to compile, then run `npm test`.
3. Commit both source changes and the updated `dist/index.js`.
4. Open a pull request targeting `main`.

## License

MIT License — see [LICENSE](LICENSE).
