---
name: document-code
description: Write or review the documentation that lives inside source files, namely identifier names, comments, Python docstrings, Terraform variable and output descriptions, and shell script headers and usage text. Reviews are read-only and report findings with suggested rewrites; files are edited only when the user asks to write, add, or fix. Use when the user asks to document, comment, or name things in code, or to review naming, comments, docstrings, or descriptions in a file, directory, or diff. Not for READMEs or other standalone docs (use document-repo).
---

# Code docs

Write or review documentation inside source files: names, comments, and the language's interface documentation (docstrings, Terraform `description`s, script usage text).

Standalone files such as READMEs, decision records, and changelogs belong to the `document-repo` skill.

## Rules

- **Review is read-only.** When the user asks to review, check, or audit, change nothing. When they ask to write, add, fix, or improve, edit the files. If the request could be either, review.
- **The repo's conventions beat this skill's defaults.** Linter config and the style of the surrounding code decide.
- **Document only what the code does.** Read the implementation before describing it. If behaviour can't be confirmed from the code, say so instead of guessing.
- **Less is better than noise.** A comment that restates the code is a finding to remove, not a gap to fill. Don't add a docstring to every private helper to look thorough.
- **Don't change behaviour.** The only code change allowed is a safe rename (see step 5).

## Steps

1. **Pick the mode**: review or write, per the first rule.

2. **Find the target**
   - Use the paths the user named.
   - Otherwise use the current changes: `git diff HEAD --name-only`. If that's empty, use the branch against its base: `git diff --name-only <base>...HEAD`.
   - If there is still nothing, ask what to look at.
   - For a diff target, judge the changed lines and the interfaces they touch, not the whole file.

3. **Detect conventions**
   - Read the reference file for each language in the target, and only those:
     - Python: [references/python.md](references/python.md)
     - Terraform: [references/terraform.md](references/terraform.md)
     - Shell: [references/shell.md](references/shell.md)
   - For other languages, apply step 4 and follow the language's standard documentation conventions.
   - Look for linter and formatter config, and read a few neighbouring files to see the existing style.

4. **Assess**, in this order:

   **Names.** A good name removes the need for a comment, so check names first.
   - The name says what the thing is or does, at the level of detail its scope needs. Short names are fine in short scopes.
   - Units and formats are in the name when the type doesn't carry them: `timeout_seconds`, `created_at_utc`.
   - Booleans read as a yes/no statement: `is_ready`, `has_children`, `enable_logging`. No negated names such as `not_disabled`.
   - No abbreviations beyond those the codebase or domain already uses.
   - One word per concept across the codebase. Don't mix `fetch`, `get`, and `retrieve` for the same operation.
   - The name still matches what the code does now.

   **Comments.**
   - Keep or add comments that explain why: intent, constraints, trade-offs, workarounds (with a link to the issue), and non-obvious consequences.
   - Remove comments that restate the code, that have gone stale, or that narrate history which belongs in version control.
   - Remove commented-out code.
   - TODOs follow the repo's format. If the repo attaches an owner or issue to them, do the same.

   **Interface documentation.** Every public interface (function, class, module, Terraform variable or output, script) states, in the language's native form:
   - what it does, in one summary line;
   - its inputs and outputs, including units, formats, and allowed values that the types don't express;
   - the errors it raises or exit codes it returns;
   - side effects and preconditions a caller must know.

   Check that existing documentation matches the signature and behaviour. Wrong documentation is worse than missing documentation, so it ranks highest.

5. **Renames**
   - In write mode, rename only what is local: local variables, private helpers, Terraform `locals`. Update every reference, and search the repo to confirm none are left.
   - Never rename these unless the user explicitly asks for it; report them as findings instead:
     - public functions, classes, modules, and CLI flags;
     - Terraform variables, outputs, resources, and modules (a resource rename without a `moved` block destroys and recreates the resource);
     - environment variables, config keys, and anything serialised or stored.

6. **Deliver**

   *Review mode.* Reply with findings, most important first: wrong or misleading documentation, then missing documentation on public interfaces, then names, then noise. For each:

   ```text
   path/to/file.py:42: <the problem in one sentence>
     Suggested: <the rewrite>
   ```

   Group repeated instances of one problem into a single finding with a list of locations. If the target is in good shape, say so briefly; don't invent findings. End by offering to apply the fixes.

   *Write mode.* Edit the files. Then run whichever of the repo's checks apply and are installed (for example `ruff check`, `terraform fmt -check` and `terraform validate`, `tflint`, `shellcheck`) and report the result. Reply with a short summary of what changed, plus anything reported rather than changed (such as public renames).

## Examples

Review of a Python function →

```text
billing/invoice.py:18: the docstring says the function returns None when the customer is missing, but it raises CustomerNotFound.
  Suggested: replace the Returns note with "Raises: CustomerNotFound: If no customer has this ID."

billing/invoice.py:31: `d` holds the grace period, and its unit isn't clear.
  Suggested: rename to `grace_period_days`.

billing/invoice.py:44: the comment "# loop over invoices" restates the code.
  Suggested: remove it.
```

Review of a Terraform variable →

```text
variables.tf:12: the description "The retention" restates the variable name and gives no unit.
  Suggested: "How long to keep log events, in days. Must be a value CloudWatch accepts, such as 30, 90, or 365."

main.tf:5: the resource `aws_s3_bucket.s3_bucket` repeats its type in its name. Renaming it needs a `moved` block, so this is reported only.
  Suggested: `aws_s3_bucket.logs` with `moved { from = aws_s3_bucket.s3_bucket, to = aws_s3_bucket.logs }`.
```
