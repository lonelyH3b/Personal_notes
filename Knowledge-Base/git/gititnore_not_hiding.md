# Why `.gitignore` Doesn't Hide Already-Tracked Files

## The Problem

I added a file or directory to `.gitignore`, but Git was still showing changes to it.

For example:

```gitignore
content/
```

Even after adding this rule, Git continued tracking files inside the `content/` directory.

## Why It Happens

`.gitignore` only applies to **untracked files**.

If a file or directory has already been added to Git and committed, Git is already tracking it. Adding it to `.gitignore` does not automatically remove it from Git's tracking.

In other words:

> `.gitignore` prevents Git from tracking new files; it does not stop tracking files that are already tracked.

## The Fix

To keep the files on the local machine but remove them from Git's tracking:

```bash
git rm -r --cached content/
```

Then commit the change:

```bash
git add .gitignore
git commit -m "Ignore content directory"
```

The `--cached` option is important because it removes the files from Git's index **without deleting them from the local filesystem**.

## For a Single File

If only one file needs to be removed from tracking:

```bash
git rm --cached filename.ext
```

Then commit the change:

```bash
git add .gitignore
git commit -m "Stop tracking ignored file"
```

## Checking Whether `.gitignore` Matches

Git provides a useful command for checking which `.gitignore` rule is responsible:

```bash
git check-ignore -v filename.ext
```

If the file is being ignored, Git will show the matching `.gitignore` rule.

## Example

Suppose the repository contains:

```text
my-project/
├── .gitignore
├── README.md
└── content/
    ├── draft.md
    └── private-notes.md
```

And `.gitignore` contains:

```gitignore
content/
```

If `content/` was already committed, Git will continue tracking it.

Run:

```bash
git rm -r --cached content/
```

Then commit:

```bash
git add .gitignore
git commit -m "Ignore content directory"
```

Now Git will stop tracking the directory while the files remain locally available.

## Key Takeaway

**`.gitignore` is not a way to untrack files.**

If something is already tracked:

1. Add it to `.gitignore`.
2. Remove it from Git's index with `git rm --cached`.
3. Commit the change.

After that, Git will leave the ignored files alone.
