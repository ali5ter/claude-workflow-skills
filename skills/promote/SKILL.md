---
name: promote
description: Runs the full release workflow for the current project. Commits any uncommitted changes, pushes to remote, creates and merges a PR if on a feature branch, determines the next semver version from conventional commits, creates an annotated git tag and GitHub release with generated notes, cleans up merged branches, and returns to a clean main. Use when the user says promote, ship, release, commit and push, tag and release, or get back to main.
allowed-tools: Bash
---

# Promote Changes

Runs the full release workflow from current working state to a tagged GitHub release on main.

Current branch: !`git branch --show-current 2>/dev/null || echo "unknown"`

Git status: !`git status --short 2>/dev/null || echo "not a git repo"`

Last tag: !`git describe --tags --abbrev=0 2>/dev/null || echo "none"`

Recent commits since last tag:

```text
!`git log $(git describe --tags --abbrev=0 2>/dev/null || echo "")..HEAD --oneline 2>/dev/null || git log --oneline -10`
```

## Step 1: Commit any uncommitted changes

If the git status above shows uncommitted or untracked changes:

1. Review the diff: `git diff` and `git diff --cached`
2. Stage all relevant changes: `git add -A`
3. Draft a conventional commit message from the changes — lead with a type prefix
   (`feat:`, `fix:`, `docs:`, `chore:`, etc.) and a concise summary
4. Commit using a heredoc so multi-line messages format correctly

If there is nothing uncommitted, skip to Step 2.

## Step 2: Push

Push the current branch to remote:

```bash
git push -u origin HEAD
```

## Step 3: Merge to main (feature branch only)

Skip this step if already on `main` (or the repo's default branch).

If on a feature branch:

1. Create a PR targeting main:

   ```bash
   gh pr create --title "<type>: <summary>" --body "$(cat <<'EOF'
   ## Summary
   <bullet points from commits>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

2. Enable auto-merge (squash preferred):

   ```bash
   gh pr merge --auto --squash
   ```

3. Poll until merged:

   ```bash
   gh pr view --json state --jq '.state'
   ```

4. Switch to main and pull:

   ```bash
   git checkout main && git pull
   ```

## Step 4: Determine next version

Analyse the commits since the last tag shown above. Apply semver rules:

- Any commit with `BREAKING CHANGE:` in the footer, or a `!` suffix on the type → **major** bump
- Any commit beginning with `feat:` or `feat(scope):` → **minor** bump
- All other commits (`fix:`, `docs:`, `chore:`, `refactor:`, etc.) → **patch** bump

If no previous tag exists, start at `v1.0.0`.

Calculate `<next-version>` from the current last tag and the rule above.

## Step 5: Tag and release

```bash
git tag -a v<next-version> -m "Release v<next-version>"
git push origin v<next-version>
gh release create v<next-version> \
  --generate-notes \
  --title "v<next-version>"
```

## Step 6: Clean up merged feature branch

If a feature branch was merged in Step 3, delete it locally and remotely:

```bash
git branch -d <feature-branch>
git push origin --delete <feature-branch>
```

## Step 7: Confirm clean state

```bash
git status
git log --oneline -5
```

Report the GitHub release URL from `gh release view v<next-version> --json url --jq '.url'`.
