---
name: draft-commit
description: Draft a succinct Conventional Commits message from the currently staged git changes and write it to scratchpad/COMMIT_MSG.txt at the repo root for the user to review. Never commits. Use when the user asks to write, draft, or suggest a commit message.
---

# Draft commit

Write a commit message for the **staged** changes to `<repo-root>/scratchpad/COMMIT_MSG.txt`. The user reviews it and commits themselves.

## Rules

- Never run `git commit`, `git add`, `git reset`, `git stash`, or anything else that changes the index, working tree, or history.
- Only describe staged changes. Ignore unstaged and untracked files.

## Steps

1. **Check preconditions**
   - `git rev-parse --show-toplevel` gives the repo root. If it fails, stop: not a git repo.
   - `git diff --cached --quiet`: if the exit code is 0, stop and tell the user nothing is staged.

2. **Read the staged changes**
   - `git diff --cached --stat`
   - `git diff --cached` (if it's very large, read the diffs of the most significant files based on the stat)
   - `git log -n 15 --oneline` to match existing types and scopes
   - If there's a commitlint config (`commitlint.config.*`, `.commitlintrc*`), follow its types, scopes, and length limits.

3. **Compose** using [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/):

   ```text
   type(scope): subject
   ```

   - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
   - Scope is optional. Use it only when the change clearly belongs to one module, or when the log uses scopes.
   - Subject: imperative mood, lowercase, no trailing period, aim for 50 characters or fewer, never more than 72.
   - **Be succinct.** The default is a subject line only. Add a body (blank line, wrapped at 72 characters, a few short lines at most) only when the *why* isn't obvious from the subject.
   - Breaking change: add `!` after the type or scope and a `BREAKING CHANGE: <what breaks>` footer.
   - Don't invent issue references.
   - If the staged changes mix unrelated concerns, write the best single message and suggest splitting the commit in your reply.

4. **Write the file**
   - `mkdir -p <root>/scratchpad`, then overwrite `<root>/scratchpad/COMMIT_MSG.txt` with the plain message, ending in a newline. No code fences.
   - `git check-ignore -q scratchpad/COMMIT_MSG.txt` (run from the root). If the file isn't ignored, warn the user.

5. **Reply briefly** with the message, the file path, and:

   ```sh
   git commit -eF scratchpad/COMMIT_MSG.txt
   ```

## Examples

Staged: a new `--json` flag in `cli/output.go` →

```text
feat(cli): add --json output flag
```

Staged: a retry loop around an HTTP call, fixing intermittent CI failures →

```text
fix(api): retry transient 503s from upstream

Upstream returns sporadic 503s under load, which failed CI runs.
```
