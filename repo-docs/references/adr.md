# Decision records (ADRs)

A decision record captures one significant choice: the situation, what was chosen, what else was considered, and what follows from it. Its reader is someone, months later, asking "why is it like this, and can I change it?"

## Detect the conventions

- Look for an existing directory: `docs/decisions/`, `docs/adr/`, `doc/adr/`, `adr/`, `decisions/`, `docs/architecture/decisions/`.
- Read the two most recent records, and any `template.md` or `0000-*.md`, for the template, numbering width, file naming, and status values.
- Check for tooling config: `.adr-dir` (adr-tools), `.log4brains.yml`.
- If there are none, create `docs/decisions/` with files named `NNNN-short-title-in-kebab-case.md`, numbered from `0001`, and use the template below.

## What deserves a record

Write one when the choice is costly to reverse, affects several parts of the system or several people, was contested, or would look odd to a newcomer. Typical cases: choosing a datastore, framework, or cloud service; an API or data format convention; a repo or deployment structure; deliberately not doing something expected.

Don't write one for a choice that is easy to change, that a linter enforces, or that is ordinary practice.

## Template

Use the repo's own if it has one. Otherwise:

```markdown
# NNNN. Short title that states the decision

- Status: proposed | accepted | rejected | deprecated | superseded by [NNNN](NNNN-title.md)
- Date: YYYY-MM-DD

## Context

The situation and the forces at play: what problem needs solving, and which
constraints and requirements apply. Facts, not the decision.

## Decision

What we will do, stated in full sentences and the active voice: "We will ...".

## Alternatives considered

Each real alternative, with the reason it was not chosen.

## Consequences

What becomes easier, what becomes harder, what we now have to do or accept,
and what would make us revisit this.
```

## Writing one

- **The reasons must come from the user or the repo.** Draw them from the conversation, linked issues and PRs, commit messages, and the code. Ask for what is missing. Never invent a rationale, an alternative that wasn't considered, or a benchmark. If the user doesn't know why an old decision was made, record that openly.
- The title names the decision, not the topic: "Use PostgreSQL for the order store", not "Database".
- One decision per record. Split a record that decides two things.
- Context states the constraints that made the choice necessary, including the non-technical ones: deadlines, team skills, cost, compliance.
- Consequences include the downsides. A record with only benefits isn't believed.
- Keep it to about a page. Link to supporting material instead of pasting it.
- Use today's date for a new record. For a decision recorded after the fact, date it when it was made if that is known, and say that it was written up later.
- Add the record to the index (`README.md` in the directory, or wherever the repo lists them), if there is one.

## Changing a decision

Accepted records are history and are not rewritten.

- To change a decision, write a new record that refers to the old one, and change only the old record's status line to `superseded by [NNNN](...)`.
- Fixing a typo or a broken link in an old record is fine. Changing its content is not.
- A `proposed` record may be edited freely until it is accepted.

## Common findings

- No context: the record states the choice with no problem or constraints.
- No alternatives, or alternatives set up to lose.
- Consequences that list only benefits.
- An accepted record whose content was edited instead of superseded (`git log -p` on the file shows it).
- Status out of date: the code has moved on, but the record still says accepted and nothing supersedes it.
- Gaps or duplicates in numbering, and records missing from the index.
- A record that is really a design document or meeting notes. Reduce it to the decision and link to the rest.
