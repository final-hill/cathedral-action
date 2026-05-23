# Cathedral PR Analysis — GitHub Action

Analyzes a pull request against your [Cathedral](https://github.com/final-hill/cathedral) requirements knowledge base: derives candidate requirements from the diff, posts a PR comment with findings, and optionally blocks merge when error-severity diagnostics are present.

## Usage

```yaml
# .github/workflows/cathedral.yml
name: Cathedral Requirements Analysis

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  cathedral:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write   # post PR comments
      statuses: write        # set commit status (Production enforcement only)
    steps:
      - uses: final-hill/cathedral-action@v1
        with:
          endpoint: ${{ vars.CATHEDRAL_ENDPOINT }}
          token: ${{ secrets.CATHEDRAL_TOKEN }}
          solution: my-solution
          enforcement-level: Developing   # or Production to block merge
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `endpoint` | ✓ | — | Your Cathedral instance URL (e.g. `https://cathedral.example.com`) |
| `token` | ✓ | — | Cathedral PAT — store as a [GitHub Secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) |
| `solution` | ✓ | — | Cathedral solution slug — find it via `cathedral list-solutions` or the Cathedral UI |
| `github-token` | — | `${{ github.token }}` | GitHub token to fetch the PR diff and post comments; the default is sufficient for most repos |
| `enforcement-level` | — | `Developing` | `Exploratory` / `Developing` = comment only. `Production` = also sets a blocking `cathedral/requirements` commit status when error-severity diagnostics are present |

## Outputs

| Output | Description |
|--------|-------------|
| `finding-count` | Number of candidate requirements derived from the PR diff |
| `error-count` | Number of error-severity diagnostics emitted by the check engine |

## Enforcement levels

| Level | PR comment | Blocks merge |
|-------|-----------|-------------|
| `Exploratory` | ✓ (advisory) | ✗ |
| `Developing` | ✓ | ✗ |
| `Production` | ✓ | ✓ (when `error-count > 0`) |

## Prerequisites

1. A running Cathedral instance — see [final-hill/cathedral](https://github.com/final-hill/cathedral)
2. A Cathedral PAT generated from **Profile → MCP Tokens** in the Cathedral UI
3. The solution slug for the project being analyzed

## How it works

1. Fetches the PR diff and head SHA from the GitHub API
2. Calls `analyze-pull-request` on your Cathedral MCP server via [`@cathedral/cli`](https://www.npmjs.com/package/@cathedral/cli)
3. Posts a Markdown comment to the PR with derived findings and diagnostics
4. If `enforcement-level` is `Production` and `error-count > 0`, sets the `cathedral/requirements` commit status to `failure`, blocking merge until resolved

## Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success — findings only, no blocking diagnostics |
| `1` | Error-severity diagnostics present (Production enforcement) |
| `2` | Authentication or connectivity failure |

## Related

- [`@cathedral/cli`](https://www.npmjs.com/package/@cathedral/cli) — the CLI that this action wraps
- [Cathedral](https://github.com/final-hill/cathedral) — the requirements knowledge base server
