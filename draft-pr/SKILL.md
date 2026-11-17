---
name: draft-pr
description: Draft a pull request title and body from the current branch's commits against its base branch, write them to scratchpad/drafts/PR_TITLE.txt and scratchpad/drafts/PR_BODY.md at the repo root, and print the GitHub CLI (gh) command to open the PR. Never creates, edits, or pushes anything. Use when the user asks to write, draft, or suggest a pull request, PR title, or PR description.
---

# Draft PR

Write a PR title and body for the **current branch** to `<repo-root>/scratchpad/drafts/PR_TITLE.txt` and `<repo-root>/scratchpad/drafts/PR_BODY.md`. The user reviews them and opens the PR themselves.

## Rules

- Never run `gh pr create`, `gh pr edit`, `gh pr merge`, `git push`, `git commit`, or anything else that changes the index, working tree, history, or remote. Read-only `gh` calls (`gh auth status`, `gh repo view`, `gh pr view`) are fine.
- Only describe commits on the branch (`<base>..HEAD`). Ignore uncommitted changes, but mention them if present.

## Steps

1. **Check preconditions**
   - `git rev-parse --show-toplevel` gives the repo root. If it fails, stop: not a git repo.
   - `git branch --show-current`. If it's empty (detached HEAD), stop.
   - `gh auth status`. If it fails, carry on using git alone and tell the user `gh` is unavailable.

2. **Find the base branch**
   - Use the base the user named, if any.
   - Otherwise: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`, falling back to `git symbolic-ref --short refs/remotes/origin/HEAD`, then `main` or `master`, whichever exists.
   - Compare against `origin/<base>` when it exists, so a stale local base doesn't add noise.
   - If the current branch is the base, or `git log --oneline <base>..HEAD` is empty, stop: nothing to propose.

3. **Check for an existing PR**
   - `gh pr view --json number,url,title,state`. If an open PR exists for this branch, note its number; the final command becomes `gh pr edit`.

4. **Read the changes**
   - `git log --reverse --format='%h %s%n%b' <base>..HEAD`
   - `git diff --stat <base>...HEAD`
   - `git diff <base>...HEAD` (if it's very large, read the diffs of the most significant files based on the stat)
   - `git status --porcelain` for uncommitted work, and `git log --oneline @{u}..HEAD` for unpushed commits (if `@{u}` fails, the branch has no upstream yet).

5. **Look for a PR template**
   - Check `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `pull_request_template.md`, `docs/pull_request_template.md` (case-insensitive), and `.github/PULL_REQUEST_TEMPLATE/`. If that directory has several templates, pick the best fit and say which.
   - If found, follow it: keep its headings and order, fill each section, remove HTML comments, and only tick checklist items the changes clearly satisfy. Leave a section's prompt unanswered rather than inventing an answer.

6. **Compose**
   - **Title:** imperative, no trailing period, 72 characters or fewer. Use Conventional Commits style (`type(scope): subject`) if the branch commits or `git log -n 15 --oneline` on the base use it. For a single-commit branch, the commit subject is usually the right title.
   - **Body without a template:** this structure is a recommendation, not a requirement. Use, drop, or add sections based on what you gathered:

     ```markdown
     ## Summary

     - What changed and why, in 1–4 bullets.

     ## Testing

     How it was verified, or "Not tested."

     ## Notes

     Breaking changes, migrations, rollout steps, or where reviewers should focus.
     ```

     A one-line docs fix may need just a sentence. A large or risky change may need more. Omit empty sections.
   - **Be succinct.** Explain intent and impact; don't restate the diff file by file.
   - Never invent test results or issue references. Add `Closes #N` only when the branch name or commit messages reference that issue.
   - If the branch mixes unrelated concerns, write the best single description and suggest splitting the PR in your reply.

7. **Write the files**
   - `mkdir -p <root>/scratchpad/drafts`, then overwrite `PR_TITLE.txt` (the title on one line, ending in a newline) and `PR_BODY.md` (the body, ending in a newline). No code fences around the content.
   - `git check-ignore -q scratchpad/drafts/PR_TITLE.txt scratchpad/drafts/PR_BODY.md` (run from the root). If they aren't ignored, warn the user.

8. **Reply briefly** with the title, the body, the file paths, any warnings (uncommitted changes, unpushed commits, no upstream, mixed concerns), and the command to run:

   ```sh
   gh pr create --base <base> --title "$(cat scratchpad/drafts/PR_TITLE.txt)" --body-file scratchpad/drafts/PR_BODY.md
   ```

   - Mention `--draft` for a draft PR.
   - If the branch has no upstream or unpushed commits, put `git push -u origin <branch>` first.
   - If a PR already exists, give this instead:

     ```sh
     gh pr edit <number> --title "$(cat scratchpad/drafts/PR_TITLE.txt)" --body-file scratchpad/drafts/PR_BODY.md
     ```

## Examples

Branch `add-json-output`, one commit `feat(cli): add --json output flag`, no template →

`PR_TITLE.txt`:

```text
feat(cli): add --json output flag
```

`PR_BODY.md`:

```markdown
## Summary

- Add a `--json` flag that prints results as JSON for scripting.

## Testing

Added unit tests for the JSON formatter.
```

Branch `fix/482-upstream-503`, three commits adding retries and a timeout config, no template →

`PR_TITLE.txt`:

```text
fix(api): retry transient 503s from upstream
```

`PR_BODY.md`:

```markdown
## Summary

- Retry upstream calls up to 3 times with backoff on 503 responses.
- Make the request timeout configurable via `API_TIMEOUT`.

Upstream returns sporadic 503s under load, which caused flaky CI runs.

## Notes

`API_TIMEOUT` defaults to the previous hard-coded 10s, so no config change is required.

Closes #482
```
