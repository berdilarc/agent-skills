# CHANGELOG

The reader uses the project and is deciding whether to upgrade and what they must do when they upgrade. The changelog tells them what changed for them, not what the developers did.

## Detect the conventions

- An existing `CHANGELOG.md` (or `CHANGES.md`, `HISTORY.md`, `NEWS.md`): keep its format, heading style, and ordering.
- Whether it is generated: look for `release-please`, `semantic-release`, `git-cliff` (`cliff.toml`), `changesets` (`.changeset/`), or `towncrier` config and CI steps. **Don't hand-edit a generated changelog.** Tell the user, and fix the input instead (commit messages, changeset files, news fragments).
- The versioning scheme and tag format: `git tag --sort=-v:refname | head`.
- If there is no changelog, use the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format below.

## Format

```markdown
# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `--json` flag to print results as JSON.

## [1.3.0] - 2026-10-02

### Changed

- **Breaking:** the `bucket_name` input is now `name`. Rename it in your module block.

### Fixed

- Retry transient 503 responses from the upstream API instead of failing.

[Unreleased]: https://github.com/OWNER/REPO/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/OWNER/REPO/compare/v1.2.0...v1.3.0
```

- Newest version first. Dates are ISO 8601.
- Sections, in this order, with empty ones left out: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.
- Keep an `Unreleased` section at the top. Only mention Semantic Versioning in the header if the project follows it.
- Fill in the comparison links from the real remote (`git remote get-url origin`), or leave them out if the host has no compare view.

## Drafting entries from history

1. Find the range: the last tag to `HEAD` (`git describe --tags --abbrev=0`), or the range the user gives.
2. Read `git log --reverse --format='%h %s%n%b' <range>`. Read the diff where a subject doesn't make the user-facing effect clear.
3. Map Conventional Commits types to sections:

   | Commit | Section |
   | --- | --- |
   | `feat` | Added, or Changed if it alters existing behaviour |
   | `fix` | Fixed |
   | `perf` | Changed |
   | `!` or `BREAKING CHANGE:` | Changed or Removed, marked **Breaking** |
   | `revert` | Remove the reverted entry if it's in the same release; otherwise Changed or Fixed |
   | security fixes | Security |
   | `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` | Left out, unless users are affected (for example, a raised minimum runtime version) |

4. Rewrite each entry for the user. Say what they can now do or what behaves differently, name the flag, function, or variable affected, and merge commits that make up one change. `fix(api): handle nil ptr in retry loop` becomes "Fix a crash when the upstream API times out during a retry."
5. For breaking changes, say what to do: the rename, the replacement, the migration step.
6. Don't invent issue or PR references. Include them only when the commits carry them, in the style the changelog already uses.

Releasing a version (renaming `Unreleased`, choosing the version number, tagging) is the user's call. Suggest the bump implied by the changes (major for breaking, minor for added, patch for fixed), and make the edit only when asked.

## Common findings

- Entries copied from commit subjects, written for developers.
- Internal changes listed (refactors, CI, test changes).
- Breaking changes not marked, or with no upgrade instruction.
- Tags with no changelog section, or sections with no tag.
- Changes merged since the last release that are missing from `Unreleased`.
- Wrong or missing dates, wrong order, broken comparison links.
