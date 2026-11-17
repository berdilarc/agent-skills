# CONTRIBUTING and development setup

The reader wants to make a change and get it accepted. They need to go from a fresh clone to a passing test run, and to know what a mergeable change looks like.

## Where it lives

- `CONTRIBUTING.md` at the root, in `.github/`, or in `docs/`. GitHub links to it from new issues and PRs.
- If the whole guide is a few lines, keep it as a "Development" section in the README instead of a separate file.
- If the repo already splits setup into its own file (`docs/development.md`), keep the split and link between the two.

## Questions it answers

1. **Setup.** Required tools with versions, and where the versions are pinned (`.tool-versions`, `.python-version`, `.nvmrc`, `required_version`). The commands to install dependencies. Required environment variables, credentials, or local services, and how to get them.
2. **The working loop.** The exact commands to build, run, test, lint, and format, including how to run a single test. Take them from the Makefile, task runner, package scripts, and CI config.
3. **Checks before pushing.** What CI runs, and how to run the same thing locally. Pre-commit hooks and how to install them.
4. **Change conventions.** Branch naming, the commit message format (link to Conventional Commits if the history uses it), and what a PR needs: tests, docs, a changelog entry, a linked issue.
5. **Review and merge.** Who reviews, how to ask for it, the merge method, and what happens after merge (release, deploy).
6. **Where to ask.** The channel or tracker for questions and bug reports. For security reports, link to `SECURITY.md` if it exists.

Leave out a section the repo has nothing to say about. A solo or internal repo may need only the first three.

## Writing it

- Order it as a newcomer meets it: setup, loop, checks, conventions, review.
- State conventions as the repo practises them. Check the last 20 or so commits and merged PRs. Don't describe a process the history doesn't show.
- Don't restate what tooling enforces (formatter rules, lint rules). Name the tool and the command.
- Don't copy the README's install steps. Link to them, or move the detail here and leave the README a pointer.
- Code of conduct, licence terms, and CLA requirements are the user's decision. Link to them if they exist; don't add them unasked.

## Common findings

- Setup steps that no longer work. CI config shows the real steps; compare the two.
- A missing prerequisite that the author's machine happened to have.
- Tool versions that disagree with the pinned ones.
- Commands for a task runner the repo has dropped (`make test` when the repo now uses `just` or `npm run`).
- Commit or branch conventions that the recent history doesn't follow. Ask which is right before changing either.
- No way to run the CI checks locally.
- Generic template text that fits no part of this repo.
