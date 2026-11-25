# .github
Zazume

## Reusable workflows

### `main-health-alert`

Posts a Slack message when `main` breaks in a repo, and another one when it is green again.

- 🔴 `main roto en <repo>`: a `push` run on `main` ends in `failure` and the previous run of the same workflow did not fail.
- 🟢 `main vuelve a verde en <repo>`: a `push` run on `main` ends in `success` right after a failed one.
- One message per breakage and per repo + workflow. Consecutive failures do not alert again.
- Each message includes the workflow, the commit (link), the commit author and a link to the run.
- Ignored: `workflow_dispatch` runs, pull requests, other branches, `cancelled` and `skipped` runs.
- The job never turns red. A missing token or a Slack error is logged as a warning.

#### Usage

Add `.github/workflows/main-health-alert.yml` to the repo. List the `name:` of every workflow that runs on `push` to `main`:

```yaml
name: main-health-alert

on:
  workflow_run:
    workflows:
      - on-push-main
    types: [completed]
    branches: [main]

permissions:
  actions: read

jobs:
  alert:
    uses: zazume/.github/.github/workflows/main-health-alert.yml@main
    with:
      slack-channel: '#alerts-critical'
    secrets:
      slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
```

#### Inputs

| Name | Required | Description |
| --- | --- | --- |
| `slack-channel` | yes | Channel name or ID, e.g. `#alerts-critical`. |

#### Secrets

| Name | Required | Description |
| --- | --- | --- |
| `slack-bot-token` | no | Slack bot token with `chat:write`. The bot must be a member of the channel. |

#### Notes

- `workflow_run` only fires when the caller file is on the default branch, so it cannot be tested from a PR branch.
- The caller needs `actions: read` to read the previous run of the workflow.
