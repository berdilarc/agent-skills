# Runbooks

The reader is handling an alert or carrying out an operational task, possibly at night, under pressure, and without having seen this system before. A runbook gives them steps they can follow without having to work out how the system behaves.

## Where they live

Follow the repo: `docs/runbooks/`, `runbooks/`, or an operations section. One file per alert or task, named after it (`high-error-rate.md`, `rotate-database-credentials.md`), with an index that lists them. If alerts link to runbooks by URL, keep file names stable, and report a rename as a finding instead of making it.

## Structure

For an alert or failure:

```markdown
# <Alert name, exactly as it appears in the alerting system>

## Meaning

What the alert indicates and what users experience while it fires.

## Impact and urgency

Who is affected and how fast this needs action.

## Diagnosis

1. A check, as an exact command or a link to a dashboard or query.
   What a healthy result looks like, and what each unhealthy result points to.

## Mitigation

Steps for each likely cause, most likely first. Exact commands.
What to expect after each step, and how to confirm recovery.

## Rollback

How to undo the mitigation if it makes things worse.

## Escalation

When to escalate, and to whom (a team or rota, not a named person).

## Background

Links: architecture, related decision records, past incidents.
```

For a routine task (rotate a secret, restore a backup, fail over), use: purpose, when to do it, prerequisites and access needed, numbered steps, verification, rollback.

## Writing one

- **Operational facts come from the user and the repo, never from assumption.** Take commands, resource names, thresholds, and dashboards from deploy config, alert rules, scripts, and Terraform. Ask for what the repo doesn't hold: escalation contacts, dashboard URLs, access procedures. Mark a gap clearly (`TODO(owner): ...`) instead of filling it with a plausible guess. A wrong step in a runbook does damage during an incident.
- Give commands in full, ready to copy, with the environment or context selection included (`--context prod-eu`, `AWS_PROFILE=...`). Show placeholders as `<like-this>` and say where the value comes from.
- Put a warning before any step that is destructive or affects users, and say what it does.
- One action per numbered step, each followed by what the reader should see.
- Start with mitigation that restores service. Root-cause investigation comes after.
- Write for someone who doesn't know the system. Link to the architecture overview for background and keep explanations out of the steps.
- Never put secrets in a runbook. Say where the secret is stored and how to get access.
- Don't run operational commands against real environments to check a runbook. Verify by reading the config, and tell the user which steps were not exercised.

## Common findings

- Commands that point at resources, namespaces, clusters, or scripts that have been renamed or removed. Compare with deploy config and the scripts directory.
- Alerts defined in the repo's alert rules that have no runbook, and runbooks for alerts that no longer exist.
- Steps that say what to do without the command: "restart the service", "check the logs".
- No verification step, so the reader can't tell whether the fix worked.
- No rollback for a risky step.
- Escalation to a named person instead of a team or rota, or to someone who has left.
- Secrets or tokens pasted into the text. Report these first: they need rotating, not only removing.
- Long explanations mixed in with the steps.
- No date or owner, where the repo's other runbooks carry them.
