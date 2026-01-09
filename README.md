# Commit Name Validator GitHub Action

This repo provides a reusable **GitHub Action** that:

- **Validates commit names** in pull requests against the required format: `[QD-{number}] {message}`
- **Posts PR comments** listing any commits that don't follow the format
- **Automatically removes** validation failure comments when all commits are fixed
- **Excludes merge commits** from validation automatically

The action is implemented as a composite action using `actions/github-script@v7`.

## Usage

Add this to your repository workflow to validate commit names on pull requests:

```yaml
name: Validate Commits
on:
  pull_request:
    branches: [ main, develop ]

permissions:
  contents: read
  pull-requests: write

jobs:
  validate-commits:
    runs-on: ubuntu-latest
    steps:
      - name: Validate commit names
        uses: ensembleip/commit-name-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Versioning / pinning

`uses: ensembleip/commit-name-action@vx` requires that this repository has a **Git tag** named `vx` (or a branch named `vx`).

- For initial testing you can use `@main`.
- For production usage, prefer pinning to a commit SHA, or use a major tag like `@v1` that you keep updated to the latest `v1.x.y`.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token used to post PR comments and fetch commits | No | `${{ github.token }}` |
| `comment-marker` | HTML comment marker to identify comments created by this action | No | `<!-- commit-name-action -->` |

## Required Format

All commits (except merge commits) must follow this format:

```
[QD-{number}] {message}
```

### Valid Examples

- ✅ `[QD-123] Fix authentication bug`
- ✅ `[QD-456] Add user profile endpoint`
- ✅ `[QD-789] Update documentation for API`

### Invalid Examples

- ❌ `Fix bug` (missing ticket reference)
- ❌ `QD-123 Fix bug` (missing square brackets)
- ❌ `[QD-] Fix bug` (missing ticket number)
- ❌ `[QD-123]` (missing message)

## Example PR Comment

When validation fails, the action posts a comment like this:

```markdown
## ❌ Commit Name Validation Failed

The following commit(s) do not follow the required format `[QD-{number}] {message}`:

- `a1b2c3d`: Fix authentication issue
- `e4f5g6h`: Update dependencies

**Required format**: `[QD-123] Your commit message here`

Please update the commit messages to follow the required format.
```

## Merge Commits

Merge commits are **automatically excluded** from validation. The action detects merge commits by:

1. Checking if a commit has more than one parent
2. Checking if the commit message starts with "Merge"

This means standard GitHub merge commits like `Merge pull request #123 from branch-name` are ignored.

## Permissions

The workflow using this action requires the following permissions:

- `contents: read` - to read repository contents
- `pull-requests: write` - to post, update, and delete comments on pull requests

## How It Works

1. **Fetches commits** from the pull request
2. **Filters out** merge commits
3. **Validates** each remaining commit against the pattern `[QD-\d+] .+`
4. If validation **fails**:
   - Creates or updates a PR comment with the list of invalid commits
   - Fails the GitHub Action check
5. If validation **passes**:
   - Removes any existing failure comment
   - Marks the check as successful

## Fixing Invalid Commits

To fix commit messages that don't follow the required format:

```bash
# Rebase interactively for the last n commits
git rebase -i HEAD~n

# In the editor, change 'pick' to 'reword' for commits you want to fix
# Save and close the editor
# Update each commit message to follow [QD-{number}] {message} format

# Force push to update the PR
git push --force-with-lease
```

## Notes

- The action only runs on pull request events. It will skip gracefully for other event types.
- The first line of each commit message is validated (multi-line commit messages are supported).
- The action uses the GitHub REST API to fetch commits and manage comments efficiently.
