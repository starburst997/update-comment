# Update PR Comment

A lightweight GitHub composite action that creates or updates pull request comments using the `gh` CLI. The action uses the first line of the comment body as a unique identifier to find and update existing comments.

## Features

- Create new PR comments or update existing ones automatically
- Uses the first line of the comment body as the identifier
- Simple and efficient implementation using `gh` CLI
- No external dependencies or JavaScript runtime required
- Returns both comment ID and URL for further workflow steps

## Usage

```yaml
- name: Update PR Comment
  uses: starburst997/update-comment@v1
  with:
    body: |
      ## Deploying my-app 🚀

      | Latest commit | ${{ github.sha }} |
      |:--------------|:------------------|
      | **Status** | ✅ Deploy successful! |
      | **Environment** | production |
```

**Note:** The `issue-number` input is optional and will automatically use the current PR number if not specified.

## Inputs

| Input          | Description                                                  | Required | Default                                   |
| -------------- | ------------------------------------------------------------ | -------- | ----------------------------------------- |
| `token`        | GitHub token for authentication                              | No       | `${{ github.token }}`                     |
| `issue-number` | Pull request or issue number (defaults to current PR number) | No       | `${{ github.event.pull_request.number }}` |
| `body`         | Comment body. The first line is used as the identifier.      | Yes      | -                                         |

## Outputs

| Output        | Description                               |
| ------------- | ----------------------------------------- |
| `comment-id`  | The ID of the created or updated comment  |
| `comment-url` | The URL of the created or updated comment |

## How It Works

1. The action extracts the first line from the `body` input
2. Searches for an existing comment on the PR that starts with that first line
3. If found, updates the existing comment with the new body
4. If not found, creates a new comment with the provided body

**Important:** Keep the first line of your comment body constant. This line serves as the unique identifier for finding and updating the comment.

## Example: Deployment Status Comment

```yaml
name: Deploy

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        id: deploy
        run: |
          # Your deployment logic here
          echo "url=https://preview.example.com" >> $GITHUB_OUTPUT

      - name: Update Deployment Comment
        if: always()
        uses: starburst997/update-comment@v1
        with:
          body: |
            ## Deployment Status 🚀

            | Latest commit | ${{ github.sha }} |
            |:--------------|:------------------|
            | **Status** | ${{ job.status == 'success' && '✅ Deployed' || '❌ Failed' }} |
            | **URL** | ${{ steps.deploy.outputs.url }} |

            [View logs](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})
```

## License

MIT
