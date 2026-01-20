# Commit Name Validator GitHub Action

This repo provides a reusable **GitHub Action** that:

- **Validates branch names** in pull requests against the required pattern: `(bug|story|task|spike)/QD-{number}-{description}`
- **Validates commit names** in pull requests against the required format: `[QD-{number}] {message}`
- **Posts PR comments** listing any violations (branch name or commit messages)
- **Automatically removes** validation failure comments when all validations pass
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
| `commit-pattern` | Regular expression pattern for validating commit message format | No | `^\\[QD-\\d+\\] .+` |
| `branch-pattern` | Regular expression pattern for validating branch name format | No | `^(bug\|story\|task\|spike)/QD-\\d+-[a-z0-9-]+` |

## Required Formats

### Branch Name Format

Branch names must follow this pattern:

```
(bug|story|task|spike)/QD-{number}-{description}
```

**Valid Examples:**
- ✅ `bug/QD-123-fix-login-issue`
- ✅ `story/QD-456-add-user-dashboard`
- ✅ `task/QD-789-update-dependencies`
- ✅ `spike/QD-321-research-new-api`

**Invalid Examples:**
- ❌ `feature/QD-123-new-feature` (wrong type, must be bug/story/task/spike)
- ❌ `bug/QD-123` (missing description)
- ❌ `bug/123-fix-issue` (missing QD- prefix)
- ❌ `QD-123-fix-issue` (missing type prefix)

### Commit Message Format

All commits (except merge commits) must follow this format:

```
[QD-{number}] {message}
```

**Valid Examples:**
- ✅ `[QD-123] Fix authentication bug`
- ✅ `[QD-456] Add user profile endpoint`
- ✅ `[QD-789] Update documentation for API`

**Invalid Examples:**
- ❌ `Fix bug` (missing ticket reference)
- ❌ `QD-123 Fix bug` (missing square brackets)
- ❌ `[QD-] Fix bug` (missing ticket number)
- ❌ `[QD-123]` (missing message)

## Example PR Comments

### When Branch Name Validation Fails

```markdown
## ❌ Validation Failed

### Branch Name Validation Failed

Branch name `feature/add-new-feature` does not match the required pattern.

**Required pattern:** `^(bug|story|task|spike)/QD-\d+-[a-z0-9-]+`

**Examples of valid branch names:**
- `bug/QD-123-fix-login-issue`
- `story/QD-456-add-user-dashboard`
- `task/QD-789-update-dependencies`
- `spike/QD-321-research-new-api`
```

### When Commit Validation Fails

```markdown
## ❌ Validation Failed

### Commit Name Validation Failed

The following commit(s) do not follow the required format `[QD-{issue-key}] {message}`:

- `a1b2c3d`: Fix authentication issue
- `e4f5g6h`: Update dependencies

**Required format:** `[QD-123] Your commit message here`

Please update the commit messages to follow the required format. You can use:
\`\`\`bash
git rebase -i HEAD~n  # where n is the number of commits to edit
# Then use 'reword' to change commit messages
\`\`\`

*Note: Merge commits are automatically excluded from this validation.*
```

### When Both Validations Fail

Both error messages will be combined in a single PR comment.

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

The action runs in four distinct steps:

### Step 1: Validate Branch Name
- Extracts the branch name from the pull request
- Validates it against the pattern `^(bug|story|task|spike)/QD-\d+-[a-z0-9-]+`
- Sets outputs for downstream steps

### Step 2: Validate Commit Names
- Fetches all commits from the pull request
- Filters out merge commits (commits with multiple parents or starting with "Merge")
- Validates each remaining commit against the pattern `^\\[QD-\\d+\\] .+`
- Sets outputs with validation results

### Step 3: Post Success Message (if all validations pass)
- Removes any existing validation failure comment
- Marks the check as successful

### Step 4: Post Failure Message (if any validation fails)
- Creates or updates a PR comment with:
  - Branch name validation errors (if applicable)
  - Commit message validation errors (if applicable)
- Fails the GitHub Action check with a descriptive error message

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

## Customization

You can customize the validation patterns by providing custom inputs:

```yaml
- name: Validate commit names
  uses: ensembleip/commit-name-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    branch-pattern: '^(feature|bugfix|hotfix)/[A-Z]+-\d+-[a-z0-9-]+$'
    commit-pattern: '^\[[A-Z]+-\d+\] .+'
```

## Notes

- The action only runs on pull request events (checked via `github.event.pull_request`).
- The first line of each commit message is validated (multi-line commit messages are supported).
- The action uses the GitHub REST API to fetch commits and manage comments efficiently.
- Both branch name and commit message patterns can be customized via inputs.
- The action uses a clean separation of concerns with distinct steps for validation and messaging.