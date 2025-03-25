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
      - uses: anecdotes-ai/github-action-pr-lint
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
| `title-regex`                               | no        | string | ^[Aa][Nn]-\d+[:\/].*                    | Regex to ensure PR title matches.                                         |
| `on-failed-regex-fail-action`               | no        | bool   | true                               | Whether the action should fail when the regex doesn't match.                                                                                                 |
| `on-failed-regex-create-review`             | no        | bool   | true                               | Whether the action should create a PR review & comment when the regex doesn't match.                                                                         |
| `on-failed-regex-request-changes`           | no        | bool   | true                               | Whether the action should request changes or just provide a comment.                                                                                         |
| `on-failed-regex-comment`                   | no        | string | "PR name does not conform with convention 😵‍💫 (PR title must start with 'AN-' followed by numbers, deliminated by a ':'' or /)" | Comment for the bot to post on PRs that fail the regex. Use %regex% to reference regex. |
| `on-succeeded-regex-dismiss-review-comment` | no        | string | "PR name conforms with convention - Great Success! 🥳" | Comment for the bot to post on PRs that succeed the regex and have their review dismissed.                                                                   |

## Developing

### Build & Package

`yarn install`

`yarn build`

`yarn package`: We package everything to a single file with Vercel's
[ncc](https://github.com/vercel/ncc). Outputs to `dist/index.js`.

## Related Reading

- [GitHub Action Metadata Syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/metadata-syntax-for-github-actions)
