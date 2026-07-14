# rlc-workflows

Reusable GitHub Actions workflows shared across every `rlc-*` repo in the Roalich hub. No repo copies CI logic — they all call these via `workflow_call`.

## Workflows

### `reusable-ci-node.yml`
Installs deps with pnpm, lints (`oxlint`), and builds. For Vite/React repos.

Caller example (`.github/workflows/ci.yml` in the consumer repo):

```yaml
name: CI
on:
  pull_request:
    branches: [develop, main]

jobs:
  ci:
    uses: klozano/rlc-workflows/.github/workflows/reusable-ci-node.yml@main
```

### `reusable-pr-conventions.yml`
Validates: branch name matches `feature/<issue-number>-<slug>`, PR title follows Conventional Commits, PR body references a ticket (`Closes #N` or `Refs #N`).

```yaml
name: PR Conventions
on:
  pull_request:
    branches: [develop, main]
    types: [opened, edited, synchronize]

jobs:
  conventions:
    uses: klozano/rlc-workflows/.github/workflows/reusable-pr-conventions.yml@main
```

### `reusable-auto-close-ticket.yml`
On push to `develop`, finds the PR that was just merged, closes the issue it references, and moves the board item to Done. Needed because GitHub's `Closes #N` only auto-closes on merges to the *default* branch (`main`), never on `develop`.

Requires a `PROJECT_TOKEN` secret in the caller repo — a personal access token with `repo` + `project` scopes (the default `GITHUB_TOKEN` can't write to a user-owned Project board). Set it once per repo: `gh secret set PROJECT_TOKEN`.

```yaml
name: Auto-close ticket
on:
  push:
    branches: [develop]

jobs:
  auto-close:
    uses: klozano/rlc-workflows/.github/workflows/reusable-auto-close-ticket.yml@main
    with:
      project-owner: klozano
      project-number: "3"
      status-field-id: PVTSSF_lAHOAH7Vi84BdU4dzhX3NUA
      done-option-id: "0374ca25"
    secrets:
      project-token: ${{ secrets.PROJECT_TOKEN }}
```
