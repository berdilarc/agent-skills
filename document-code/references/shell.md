# Shell

## Detect the conventions

- The shebang decides the dialect: `#!/usr/bin/env bash` or `#!/bin/sh`. Don't suggest bash-only syntax for a POSIX `sh` script.
- `.shellcheckrc` and `.editorconfig`.
- How existing scripts write their header, usage text, and function comments.

## Naming

- Functions and local variables: `lower_snake_case`.
- Exported variables, environment variables, and constants: `UPPER_SNAKE_CASE`. Declare constants `readonly`.
- Don't reuse names the shell or environment owns: `PATH`, `HOME`, `USER`, `IFS`, `PWD`.
- Give positional parameters a name at the top of the script or function, instead of using `$1` and `$2` throughout:

  ```bash
  local source_dir="$1"
  local target_bucket="$2"
  ```

- Script file names: `lower-kebab-case` or `lower_snake_case`, matching the repo. Executables on `PATH` have no `.sh` extension; libraries meant to be sourced keep it.

Environment variable names are an interface. Report a bad one; don't rename it unless asked.

## Header comment

Every script starts, after the shebang, with a comment that gives:

- what the script does, in one or two lines;
- usage, if there is no `usage` function that prints it;
- the environment variables it reads, and which are required;
- external tools it needs beyond the standard ones (`jq`, `aws`, `gh`);
- side effects worth a warning: deletes files, needs root, changes remote state.

```bash
#!/usr/bin/env bash
#
# Upload the nightly database dump to the backup bucket and delete local
# dumps older than seven days.
#
# Environment:
#   BACKUP_BUCKET  (required) Name of the target S3 bucket.
#   DUMP_DIR       Directory holding the dumps. Default: /var/backups/db.
#
# Requires: aws, gzip
```

Keep it short. Change history and author lists belong in version control.

## Usage text

- A script that takes arguments or flags has a `usage` function, printed for `-h`/`--help` (exit 0, to stdout) and on a bad invocation (non-zero exit, to stderr).
- The usage text matches the argument parser exactly: every flag handled is listed, and every flag listed is handled. Check both directions, since this is the most common mismatch.
- Show the synopsis, each option with its argument and default, and one example if the invocation isn't obvious.

## Function comments

Comment a function when its name and argument names don't say everything. Use the repo's format; otherwise:

```bash
# Wait until the service answers its health check.
# Arguments:
#   $1 - Base URL of the service.
#   $2 - Timeout in seconds. Default: 60.
# Returns:
#   0 if the service became healthy, 1 on timeout.
```

Mention globals the function reads or changes, and whether it writes its result to stdout.

## Comments

- Explain the non-obvious: why a `set` option is switched off for a section, why a command's failure is ignored (`|| true`), what a dense `sed`, `awk`, or parameter expansion does, why a `sleep` or retry is there.
- Every `# shellcheck disable=SCxxxx` gets a reason on the same or the previous line.
- Error messages go to stderr and say what failed and what to do about it.

## Checks after editing

`shellcheck <scripts>` if installed, and `shfmt -d <scripts>` if the repo uses it. Compare the usage text against the argument parser once more after any change to either.
