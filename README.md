# Pull Request Linter

A fast 🔥 TypeScript GitHub Action to ensure that your PR title
matches a given regex.

Supports the following feedback mechanisms 🛠:

- 🤖 Review, request/dismiss changes, and comment with bot
- ❌ Fail action

## Usage

Create a workflow definition at `.github/workflows/<my-workflow>.yml` with
something like the following contents:

```yaml
name: PR Lint

on:
  pull_request:
    # By default, a workflow only runs when a pull_request's activity type is opened, synchronize, or reopened. We
    # explicity override here so that PR titles are re-linted when the PR text content is edited.
    #
    # Possible values: https://help.github.com/en/actions/reference/events-that-trigger-workflows#pull-request-event-pull_request
    types: [opened, edited, reopened, synchronize]

jobs:
  pr-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: morrisoncole/pr-lint-action@v1.7.1
        with:
          # Note: if you have branch protection rules enabled, the `GITHUB_TOKEN` permissions
          # won't cover dismissing reviews. Your options are to pass in a custom token
          # (perhaps by creating some sort of 'service' user and creating a personal access
          # token with the correct permissions) or to turn off `on-failed-regex-request-changes`
          # and use action failure to prevent merges instead (with
          # `on-failed-regex-fail-action: true`). See:
          # https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token
          # https://docs.github.com/en/rest/pulls/reviews#dismiss-a-review-for-a-pull-request
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          title-regex: "#[eE][xX]-[0-9]+"
          on-failed-regex-fail-action: false
          on-failed-regex-create-review: true
          on-failed-regex-request-changes: false
          on-failed-regex-comment:
            "This is just an example. Failed regex: `%regex%`!"
          on-succeeded-regex-dismiss-review-comment:
            "This is just an example. Success!"
```

## Options

| Option                                      | Required? | Type   | Default Value                      | Description                                                                                                                                                   |
| ------------------------------------------- | --------- | ------ | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `repo-token`                                | yes       | string | N/A                                | [About the `GITHUB_TOKEN` secret](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#about-the-github_token-secret).           |
| `title-regex`                               | yes       | string | ".\*"                              | A JavaScript regex to test the title of each Pull Request against. Allows anything by default.                                                                |
| `on-failed-regex-fail-action`               | no        | bool   | false                              | If the regex fails, sets the action status to failed. When the action exits it will be with an exit code of 1.                                                |
| `on-failed-regex-create-review`             | no        | bool   | true                               | If the regex fails, uses the GitHub review mechanism to submit a review. The review type is determined by `on-failed-regex-request-changes`.                  |
| `on-failed-regex-request-changes`           | no        | bool   | true                               | Uses 'Request Changes' when creating a review. Otherwise, uses 'Comment'. _Note:_ if `on-failed-regex-create-review` is `false`, this won't do anything.      |
| `on-failed-regex-comment`                   | no        | string | "PR title failed to match %regex%" | Comment for the bot to post on PRs that fail the regex (or the console output if `on-failed-regex-create-review` is `false`). Use %regex% to reference regex. |
| `on-succeeded-regex-dismiss-review-comment` | no        | string | "All good!"                        | The message to post as a comment when the regex succeeds after previously failing.                                                                            |

## Developing

### Build & Package

`yarn install`

`yarn build`

`yarn package`: We package everything to a single file with Vercel's
[ncc](https://github.com/vercel/ncc). Outputs to `dist/index.js`.

## Related Reading

- [GitHub Action Metadata Syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/metadata-syntax-for-github-actions)
