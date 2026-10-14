# Git: Clearing Commit History Using an Orphan Branch

## 1. Overview

Git stores project history as a sequence of commits. Sometimes, we may want to remove the existing commit history while keeping the current project files.

Git provides the `--orphan` option to create a new branch that has no parent commit. By committing the current files on this branch, we create a fresh history that starts with a single initial commit.

**Use case:** Resetting a personal repository when the previous commit history is no longer needed.

**Important:** This method replaces the history of the target branch. It does not automatically erase every historical reference, clone, fork, or copy of the old commits.

## 2. Commands and Their Explanations

### Step 1: Navigate to the repository

```bash
cd /path/to/your-repository
```

Move into the local Git repository where you want to reset the history.

### Step 2: Create an orphan branch

```bash
git checkout --orphan fresh-start
```

Creates and switches to a new branch named `fresh-start` without a parent commit.

Unlike a normal branch, which points to a commit in the existing history, an orphan branch begins an independent history.

Depending on the Git version and repository state, existing files may remain in the working directory or index. Review the staged changes before committing.

### Step 3: Stage the current files

```bash
git add -A
```

Stages additions, modifications, and deletions in the working tree.

Before running this command, verify that the repository does not contain secrets, generated files, or other content that should not be committed.

### Step 4: Create a new initial commit

```bash
git commit -m "Initial commit"
```

Creates the first commit in the new history. The commit contains the staged snapshot of the current files.

Previous commits are not parents of this commit.

### Step 5: Delete the old local branch

```bash
git branch -D main
```

Deletes the local `main` branch reference.

**Warning:** `-D` forces deletion even if Git considers the branch unmerged. Do not run it before confirming that the new branch contains the intended files and commit.

### Step 6: Rename the new branch

```bash
git branch -m main
```

Renames the current branch from `fresh-start` to `main`.

The branch now has the expected name but retains its new, independent history.

### Step 7: Replace the remote branch history

```bash
git push --force origin main
```

Force-pushes the new `main` history to GitHub, replacing the remote `main` branch history.

This is the destructive step: commits that were reachable only through the previous remote branch may no longer appear in the normal branch history.

A protected branch may reject the push, and other branches or tags may still reference old commits.

## 3. How the History Changes

Before the reset:

```text
A ── B ── C ── D  (main)
```

After creating the orphan branch and making a new initial commit:

```text
N  (fresh-start)
```

After renaming and force-pushing:

```text
N  (main)
```

Here, `N` contains the current project snapshot, but it has no parent relationship with commits A, B, C, or D.

The new history is independent of the old one.

## 4. Advantages

- **Clean history:** Removes the previous commit sequence from the rewritten branch.
- **Preserves current files:** Keeps the current project snapshot, provided the intended files are staged and committed.
- **Simple workflow:** Requires only a few Git commands and no history-rewriting utility.
- **Useful for personal repositories:** Appropriate when old commit history is unnecessary and a fresh starting point is desired.
- **Fresh public timeline:** GitHub's normal commit history for the rewritten branch starts from the new initial commit.

## 5. Disadvantages and Risks

### A. Dangerous in team environments

Suppose a teammate has already cloned the repository:

```text
Teammate's local main: A ── B ── C ── D
GitHub's new main:     N
```

The teammate's local branch still refers to the old history. Their local repository does not automatically become consistent with the rewritten remote history.

If they push old commits, attempt a merge, or create a pull request from an outdated branch, they may reintroduce old history or encounter confusing conflicts and unrelated histories.

**Lesson:** Coordinate with every contributor before rewriting shared history.

### B. Force-push can overwrite remote work

The `--force` option does not verify that the remote branch still contains the expected history. If another contributor pushes new commits after you last fetched, your force-push can remove those commits from the branch.

A safer alternative for an intentional history rewrite is:

```bash
git push --force-with-lease origin main
```

This normally rejects the push if the remote branch has advanced beyond the remote-tracking state Git expects. It is safer, but it does not eliminate every risk.

### C. Commit history is not the same as all stored copies

Replacing `main` does not guarantee the old commits have been erased everywhere. They might remain referenced by:

- Other remote branches and tags.
- Other developers' local clones.
- Forks and backups.
- Pull requests, cached views, or other GitHub references.

### D. Auditability is lost

Historical commits help teams understand why code changed, identify regressions, review decisions, and trace responsibility.

Replacing the entire history removes that useful context from the rewritten branch.

### E. Current files can still contain secrets

A fresh initial commit does not clean the current snapshot automatically. If an API key is still present in a tracked file, it will be committed again.

If a secret was exposed, revoke or rotate it first. Rewriting history is not a substitute for revoking compromised credentials.

## 6. When Should You Use This Method?

| Situation | Recommendation |
|---|---|
| Personal repository with no need for old history | Reasonable option |
| Disposable practice repository | Usually acceptable |
| Shared production repository | Avoid unless coordinated and approved |
| Repository with important audit history | Prefer targeted history cleanup |
| Exposed API key or password | Revoke the credential and clean affected history |
| Multiple active branches and tags | Audit all references before rewriting |

## 7. Practical Safety Checklist

Before resetting history:

- [ ] Create a backup or separate clone.
- [ ] Confirm the correct repository and target branch.
- [ ] Check the current working tree and staged files.
- [ ] Remove secrets and sensitive files from the current snapshot.
- [ ] Ensure the new initial commit contains all intended files.
- [ ] Check for collaborators, open pull requests, branches, and tags.
- [ ] Obtain team approval if the repository is shared.
- [ ] Coordinate with teammates before force-pushing.
- [ ] Verify the remote history after the push.

## 8. Key Takeaway

The orphan-branch method is a straightforward way to start a new Git history while retaining the current project snapshot. Its simplicity makes it useful for personal projects, but its destructive force-push step makes it risky in collaborative environments.

**Remember:** A Git branch is a reference to a commit, not a separate copy of the project. Rewriting a branch changes which commits it points to; it does not automatically erase every copy of the previous history.