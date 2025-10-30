# 💬 Update PR Comment 🎉

A lightweight GitHub composite action that creates or updates pull request comments using the `gh` CLI 🚀. The action uses the first line of the comment body as a unique identifier to find and update existing comments ✨.

## ✨ Features 🌟

- 🆕 Create new PR comments or update existing ones automatically
- 🔗 Automatically updates linked issue comments (when running in PR context)
- 🎯 Uses the first line of the comment body as the identifier
- ⚡ Simple and efficient implementation using `gh` CLI
- 📦 No external dependencies or JavaScript runtime required
- 📤 Returns comment ID, URL, and body for further workflow steps

## 📖 Usage 🛠️

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

**Note:** 📝 The `issue-number` input is optional and will automatically use the current PR number if not specified.

## 🎛️ Inputs ⚙️

### 🔑 Core Inputs

| Input                 | Description                                                                  | Required | Default                                   |
| --------------------- | ---------------------------------------------------------------------------- | -------- | ----------------------------------------- |
| `token`               | GitHub token for authentication                                              | No       | `${{ github.token }}`                     |
| `issue-number`        | Pull request or issue number (defaults to current PR number)                 | No       | `${{ github.event.pull_request.number }}` |
| `body`                | Comment body. The first line is used as the identifier.                      | Yes      | -                                         |
| `update-linked-issue` | Whether to also update linked issue comments (searches for connected issues) | No       | `true`                                    |

### 📋 Template Inputs (optional) 🎨

**📌 Template Selection** (choose one or use custom `body`) 🎭:

| Input                 | Description             | Default |
| --------------------- | ----------------------- | ------- |
| `template-pr-deploy`  | Use PR deploy template  | `false` |
| `template-pr-cleanup` | Use PR cleanup template | `false` |

**🔧 Template Parameters** ⚙️:

| Input                        | Description                              | Default                        |
| ---------------------------- | ---------------------------------------- | ------------------------------ |
| `template-repository-name`   | Repository name for template             | `github.event.repository.name` |
| `template-sha`               | Commit SHA for template                  | `github.sha`                   |
| `template-status`            | Status: 'success', 'progress', or 'fail' | -                              |
| `template-version`           | Version for template                     | -                              |
| `template-future-version`    | Future version for template              | -                              |
| `template-environment`       | Environment for template                 | -                              |
| `template-url`               | URL for template                         | -                              |
| `template-pattern`           | Pattern for cleanup template             | -                              |
| `template-versions-markdown` | Versions markdown for cleanup template   | -                              |
| `template-logs-url`          | Logs URL for template                    | GitHub Actions run URL         |

## 📤 Outputs 📊

| Output         | Description                                |
| -------------- | ------------------------------------------ |
| `comment-id`   | The ID of the created or updated comment   |
| `comment-url`  | The URL of the created or updated comment  |
| `comment-body` | The body of the created or updated comment |

## 🔍 How It Works ⚙️

1. The action extracts the first line from the `body` input
2. Searches for an existing comment on the PR that starts with that first line
3. If found, updates the existing comment with the new body
4. If not found, creates a new comment with the provided body
5. If running in a PR context with `update-linked-issue: true`:
   - Searches for linked issues using GitHub's timeline API (for issues linked with "closes #X", "fixes #X", etc.)
   - Falls back to parsing the PR body for `#XXX` references
   - Updates or creates the same comment in the linked issue (if found)

**Important:** ⚠️ Keep the first line of your comment body constant. This line serves as the unique identifier for finding and updating the comment. 🎯

## 💡 Example: Deployment Status Comment 🚀

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

## 🎨 Templates 📝

Instead of providing a custom `body`, you can use built-in templates for common use cases.

### 🚀 Deploy Template 📦

```yaml
- name: Update Deploy Comment
  uses: starburst997/update-comment@v1
  with:
    template-pr-deploy: true
    template-status: success # or 'progress', 'fail'
    template-version: v1.2.3
    template-future-version: v1.3.0
    template-environment: production
    template-url: https://app.example.com
```

**📋 Output:**

```markdown
## Deploying my-app 🚀

| Latest commit      | abc1234                 |
| :----------------- | :---------------------- |
| **Status**         | ✅ Deploy successful!   |
| **Version**        | `v1.2.3`                |
| **Future Version** | `v1.3.0`                |
| **Environment**    | `production`            |
| **URL**            | https://app.example.com |

[View logs](https://github.com/owner/repo/actions/runs/123456)
```

### 🧹 Cleanup Template 🗑️

```yaml
- name: Update Cleanup Comment
  uses: starburst997/update-comment@v1
  with:
    template-pr-cleanup: true
    template-environment: staging
    template-pattern: "pr-*"
    template-future-version: v1.3.0
    template-versions-markdown: |
      - v1.2.1
      - v1.2.0
```

**📋 Output:**

```markdown
## Deploying my-app 🚀

| Status          | 🧹 **Cleaned up** |
| :-------------- | :---------------- |
| **Environment** | `staging`         |
| **Pattern**     | `pr-*`            |
| **Version**     | `v1.3.0`          |

**✅ Cleanup completed:** 🎊

- ✓ 🏷️ Git tags deleted
- ✓ 🐳 Docker images removed from registry
- ✓ ⎈ Helm charts removed from registry
- ✓ 🗂️ GitHub deployments deleted

**🔢 Versions cleaned:** 🧼

- v1.2.1
- v1.2.0
```

## 🔗 Linked Issue Updates 🎯

By default, when running in a PR context, this action will automatically search for and update linked issues with the same comment. This is useful for keeping issue threads updated with deployment status or other PR-related information.

### 🔎 How Linked Issues Are Found 🕵️

1. **📅 Timeline API**: First checks for issues formally linked via GitHub keywords like "closes #123", "fixes #456", "resolves #789" 🔗
2. **📄 PR Body Parsing**: If no formal link exists, parses the PR description for any `#XXX` references and uses the first one found 🔍
3. **✅ Validation**: Verifies the found number is an actual issue (not another PR) before updating 🎯

### 🚫 Disabling Linked Issue Updates ⛔

If you don't want comments to be mirrored to linked issues, set `update-linked-issue: false`:

```yaml
- name: Update PR Comment
  uses: starburst997/update-comment@v1
  with:
    update-linked-issue: false
    body: |
      ## This comment will only appear in the PR
      Not in any linked issues!
```

## 📜 License 📄

MIT 🎉
