# Releasing `commit-name-action`

GitHub Actions references use Git refs. For example, `uses: ensembleip/commit-name-action@v1` requires a tag (or branch) named `v1` in this repository.

## Recommended approach (major tag + semver tags)

Create a semver tag for the release, and move the major tag to the same commit:

```bash
git checkout main
git pull

# Create an immutable semver tag for this release
git tag -a v1.0.0 -m "commit-name-action v1.0.0"
git push origin v1.0.0

# Create (or move) the floating major tag
git tag -f v1
git push -f origin v1
```

Now downstream repos can use:

- `ensembleip/commit-name-action@v1` (tracks latest v1)
- `ensembleip/commit-name-action@v1.0.0` (fixed version)
- `ensembleip/commit-name-action@<commit SHA>` (most secure pin)

## Testing before release

Before tagging a release, test the action in a real PR:

1. Create a test repository or use an existing one
2. Reference the action using a branch or commit SHA:
   ```yaml
   uses: ensembleip/commit-name-action@main
   # or
   uses: ensembleip/commit-name-action@abc123def456
   ```
3. Create a test PR with both valid and invalid commit names
4. Verify that:
   - Invalid commits trigger a failure comment
   - Valid commits remove any existing failure comment
   - Merge commits are properly excluded

## Version conventions

- **v1.x.x**: Production-ready releases
- **v1**: Floating tag that always points to the latest v1.x.x release
- Use semantic versioning for all releases
