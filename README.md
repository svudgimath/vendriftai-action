# VendriftAI

Dependabot for APIs: VendriftAI follows the API clients your code depends on, upgrades them, and opens pull requests
that migrate your code to the new version, checked by your build and tests before they open. It runs inside your own
GitHub Actions, with your own Anthropic API key. Your code never leaves your CI.

This repository holds the compiled Action (v1.0.1).

## Use it

```yaml
# .github/workflows/vendriftai.yml
name: VendriftAI
on:
  schedule:
    - cron: '0 8 * * 1'          # Mondays, 08:00 UTC
  workflow_dispatch:             # "Run workflow" scans right away
  issue_comment:                 # a "/vendriftai" answer on an issue or PR it opened
    types: [created]

jobs:
  vendriftai:
    if: >-
      github.event_name != 'issue_comment' ||
      startsWith(github.event.comment.body, '/vendriftai')
    runs-on: ubuntu-latest
    concurrency:
      group: vendriftai
      cancel-in-progress: false
    permissions:
      contents: write            # push the fix branch
      pull-requests: write       # open the PR
      issues: write              # track what no PR could cover, and read answers
      checks: read               # read CI results on the fix branch
      statuses: read
      actions: read              # read failing CI logs, to repair a fix
    steps:
      - uses: svudgimath/vendriftai-action@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Only for an Anthropic key that isn't scoped to a workspace:
          # anthropic-workspace-id: ${{ vars.ANTHROPIC_WORKSPACE_ID }}
```

Then:

1. Add your Anthropic API key as the repository secret `ANTHROPIC_API_KEY`.
2. In **Settings → Actions → General → Workflow permissions**, check **Allow GitHub Actions to create and approve pull
   requests**. Without it, VendriftAI can't open its PRs.
3. To have VendriftAI build and test its fixes before opening a PR, add the setup steps for your languages before it
   (for example `actions/setup-python`, `actions/setup-java`). Node.js is always there.

Run it from **Actions → VendriftAI → Run workflow**.

© VendriftAI. All rights reserved. You may use this Action in your repositories; redistribution or modification is not
permitted. Third-party notices: `index.mjs.LEGAL.txt`.
