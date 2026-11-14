---
name: repo-docs
description: Write, update, or review a repository's standalone documentation (README, architecture decision records, CONTRIBUTING and development setup guides, CHANGELOG, architecture overviews, and runbooks), or audit a repo to find which documents are missing, stale, or unnecessary. Reviews and audits are read-only and report findings; files are written only when the user asks to write, add, update, or fix. Use when the user asks to write or review a README, ADR, decision record, contributing guide, changelog, architecture doc, or runbook, or asks what documentation a repo needs. Not for comments or docstrings inside source files (use code-docs).
---

# Repo docs

Write, review, or audit the documents that explain a repository to the people who use, run, and change it.

Documentation inside source files (names, comments, docstrings, Terraform descriptions) belongs to the `code-docs` skill. Commit messages and PR descriptions belong to `draft-commit` and `draft-pr`.

## Rules

- **Review and audit are read-only.** When the user asks to review, check, or audit, or asks what is missing, change nothing. When they ask to write, add, update, or fix, edit the files. If the request could be either, review.
- **Every claim is checked against the repo.** Commands, paths, flags, versions, and config keys in a document must exist as written. Read the code, manifests, and CI config before stating how something works. Leave out what can't be confirmed, and tell the user what was left out.
- **The repo's conventions beat this skill's defaults.** Keep the existing structure, heading style, tone, and file locations unless they are the problem.
- **Only the documents the repo needs.** Don't create or recommend a document because a checklist has it. An empty or boilerplate document is worse than none.
- **One home per fact.** Link to the place that owns the detail instead of copying it. Copies drift.
- **Never invent** decisions, reasons, owners, contact details, benchmarks, or history. Ask, or leave a clearly marked gap for the user to fill.

## Steps

1. **Pick the entry point**
   - *Write*: create or update one document.
   - *Review*: assess one document.
   - *Audit*: assess the repo's documentation as a whole ("what docs are missing?", "review the docs in this repo").

2. **Learn the repo**, in proportion to the task:
   - the top-level tree, and any `docs/` directory;
   - manifests and build files (`pyproject.toml`, `package.json`, `go.mod`, `*.tf`, `Makefile`, `Dockerfile`) for the real install, build, test, and run commands;
   - CI config, to see what is actually run and enforced;
   - existing documents, for structure and tone;
   - `git log` and tags when history matters (changelog, decisions).

3. **Read the reference for each document type involved**, and only those:

   | Document | Reference |
   | --- | --- |
   | README | [references/readme.md](references/readme.md) |
   | Decision record (ADR) | [references/adr.md](references/adr.md) |
   | CONTRIBUTING, development setup | [references/contributing.md](references/contributing.md) |
   | CHANGELOG | [references/changelog.md](references/changelog.md) |
   | Architecture overview | [references/architecture.md](references/architecture.md) |
   | Runbook | [references/runbook.md](references/runbook.md) |

   For other document types, apply the rules above and the checks in step 5.

4. **Write** (write entry point)
   - Start from the reader and what they came to do. Put what most readers need first.
   - Use commands exactly as they work in this repo, in copyable code blocks.
   - Match the format of existing documents. Otherwise use Markdown with one `#` title, sentence-case headings, and relative links between files in the repo.
   - When updating, change what is needed and keep the rest. Don't restructure a document that was only meant to gain a section.
   - Reply with a short summary, a list of the claims you verified by running or reading, and the gaps left for the user.

5. **Review** (review entry point). Check in this order, since the earlier problems cost readers more:
   1. **Accuracy.** Commands, paths, flags, versions, links, and described behaviour match the repo. Run cheap, read-only commands to confirm when that is safe (`--help`, `make -n`); otherwise confirm by reading.
   2. **Completeness.** The document answers the questions its reader arrives with. The reference lists them per type.
   3. **Structure.** The most needed content comes first, headings can be scanned, and nothing is duplicated from another document.
   4. **Clarity.** Plain, direct sentences. Terms are defined or linked. No filler and no marketing.

   Reply with findings, most important first:

   ```text
   README.md:23: <the problem in one sentence>
     Suggested: <the rewrite or the action>
   ```

   If the document is in good shape, say so briefly; don't invent findings. End by offering to apply the fixes.

6. **Audit** (audit entry point)
   - Classify the repo: library, service, CLI tool, Terraform module, infrastructure/live config, internal tool or script collection, or a mix.
   - Compare what exists with what that kind of repo needs:

     | Document | Needed when |
     | --- | --- |
     | README | Always. |
     | LICENSE | The repo is public, or shared outside the team. |
     | CONTRIBUTING / dev setup | Others contribute, or setup takes more than the README's few lines. |
     | CHANGELOG | The repo publishes versions that others consume (library, CLI, module). Not needed if releases are generated elsewhere, such as GitHub Releases. |
     | Decision records | Choices have been made that a newcomer would question or might undo. |
     | Architecture overview | More than a few components, or structure that the tree doesn't reveal. |
     | Runbooks | Something is deployed and someone is on call for it. |
     | Usage docs beyond the README | The README's usage section has outgrown it. |

   - For each existing document, do a quick accuracy and staleness check: compare it with the current tree and manifests, and compare its last change (`git log -1 --format=%cs -- <file>`) with the activity in the code it describes.
   - Reply with three short lists: **missing** (and why this repo needs it), **stale or wrong** (with `file:line`), and **unnecessary or duplicated**. Then recommend an order to tackle them, and offer to start on the first.

## Examples

Audit of a small Terraform module repo →

```text
Missing
- CHANGELOG.md: the module is tagged (v1.2.0, v1.3.0) and consumed by other repos, but nothing tells consumers what changed between versions.

Stale or wrong
- README.md:14: the usage example passes `bucket_name`, but the variable was renamed to `name` in v1.3.0.
- README.md:31: the inputs table lacks `enable_versioning`. It is generated by terraform-docs and needs regenerating.

Unnecessary
- docs/ARCHITECTURE.md: three lines that repeat the README's first paragraph. Remove it or fold it in.

Suggested order: fix the README example (it breaks copy-paste for new users), regenerate the inputs table, then start a changelog.
```

Review of a README →

```text
README.md:9: the quick start runs `make dev`, but the Makefile has no `dev` target; `make run` starts the server.
  Suggested: replace with `make run`.

README.md:1: the README never says what the service does. It opens with build badges and installation.
  Suggested: add one or two sentences under the title that say what it is and who uses it.
```
