# gjs-create-release
> Create GitHub Releases and upload release assets from a GitHub Actions workflow.

## Overview
This JavaScript (Node.js 12) action wraps [`softprops/action-gh-release`](https://github.com/softprops/action-gh-release) to create or update GitHub Releases, upload binary/artifact files, and optionally generate release notes automatically. Use it at the end of your CI pipeline to publish versioned releases with changelogs and attached assets.

## Usage

```yaml
name: Release
on:
  push:
    tags:
      - 'v*'
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build artifacts
        run: make build

      - name: Create GitHub Release
        id: release
        uses: partior-libs/gjs-create-release@main
        with:
          tag_name: ${{ github.ref_name }}
          name: "Release ${{ github.ref_name }}"
          body_path: CHANGELOG.md
          draft: false
          prerelease: false
          files: |
            dist/*.tar.gz
            dist/*.zip
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Print release URL
        run: echo "Released at ${{ steps.release.outputs.url }}"
```

### Cross-repository release

```yaml
      - name: Release to another repo
        uses: partior-libs/gjs-create-release@main
        with:
          repository: my-org/my-other-repo
          token: ${{ secrets.CROSS_REPO_PAT }}
          tag_name: v1.0.0
          generate_release_notes: true
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag_name` | ❌ | `$GITHUB_REF` | Git tag for the release |
| `name` | ❌ | tag name | Human-readable release title |
| `body` | ❌ | — | Markdown release notes (inline) |
| `body_path` | ❌ | — | Path to a file containing release notes |
| `draft` | ❌ | `false` | Create as a draft (unpublished) release |
| `prerelease` | ❌ | `false` | Mark as a pre-release |
| `files` | ❌ | — | Newline-delimited glob patterns for assets to upload |
| `fail_on_unmatched_files` | ❌ | `false` | Fail if a `files` glob matches nothing |
| `repository` | ❌ | current repo | Target repository in `owner/repo` format |
| `token` | ❌ | `${{ github.token }}` | GitHub token for API access |
| `target_commitish` | ❌ | — | Commit SHA or branch the tag points to |
| `discussion_category_name` | ❌ | — | Create a linked discussion in this category |
| `generate_release_notes` | ❌ | `false` | Auto-generate release notes from merged PRs |
| `append_body` | ❌ | `false` | Append `body`/`body_path` to auto-generated notes |

## Outputs

| Output | Description |
|--------|-------------|
| `url` | HTML URL of the created/updated release |
| `id` | Numeric release ID |
| `upload_url` | Asset upload URL for the release |
| `assets` | JSON array of uploaded asset metadata |

## Prerequisites
- A tag must exist (or be pushed) before this step runs when `tag_name` is set to a tag ref.
- `token` must have `contents: write` permission to create releases and upload assets.
- For `discussion_category_name`, the token also needs `discussions: write`.

## Contributing
Commit message format: `git commit -m "<TICKET_NUMBER> <COMMIT_MESSAGE>"`

## License
See [LICENSE](LICENSE). Upstream source: [`softprops/action-gh-release`](https://github.com/softprops/action-gh-release).
