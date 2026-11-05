# Docs Forwardport Action

GitHub Action that forwardports documentation changes from versioned folders to main folders in Docusaurus projects.

## Overview

This action is the inverse of [docs-backport-action](https://github.com/loft-sh/docs-backport-action). When contributors edit docs via "Edit this page" links (which point to versioned sources), this action propagates those changes to the main documentation folder.

## Usage

```yaml
name: Forwardport Documentation

on:
  pull_request_target:
    types: ["labeled", "closed"]
    paths:
      - 'vcluster_versioned_docs/**'
      - 'platform_versioned_docs/**'

jobs:
  forwardport:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: loft-sh/docs-forwardport-action@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## How It Works

1. Trigger: Add the `forwardport-main` label to a PR that modifies versioned docs
2. Detection: Action detects changes in `vcluster_versioned_docs/` or `platform_versioned_docs/`
3. Mapping: Strips version folder prefix to map to main folder path
4. PR Creation: Creates a new PR with forwardported changes after merge

A file with no matching place in the main folder isn't created. That covers a modified or renamed file that doesn't exist at the same path in the main folder, and an added file whose folder the main folder lacks. After a docs restructure those pages live at different paths, and copying them by path would bring them back as duplicates. The forwardport PR lists them under "Manual review needed". When nothing else was copied, the action comments the list on the original PR instead. Retries update that comment instead of posting a new one.

If any file fails to process, for example because a lookup returns an error other than 404, the run fails and no forwardport PR is opened. Fix the cause and rerun the job.

## Folder Mapping

| Source | Target |
|--------|--------|
| `vcluster_versioned_docs/version-X.Y.Z/path/file.mdx` | `vcluster/path/file.mdx` |
| `platform_versioned_docs/version-X.Y.Z/path/file.mdx` | `platform/path/file.mdx` |

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github_token` | GitHub token for API requests | Yes | `${{ github.token }}` |

## Development

```bash
npm install
npm run build
npm test
```

## License

MIT
