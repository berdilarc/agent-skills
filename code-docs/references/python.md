# Python

## Detect the conventions

- Docstring style, in order of authority:
  1. `[tool.ruff.lint.pydocstyle] convention` in `pyproject.toml` or `ruff.toml`, or `[pydocstyle] convention` in `setup.cfg`, `tox.ini`, or `.pydocstyle`.
  2. The style most existing docstrings use: `Args:` (Google), `Parameters` with a dashed underline (NumPy), or `:param x:` (Sphinx/reST).
  3. Otherwise Google style.
- Which rules are enforced: `select`/`ignore` under `[tool.ruff.lint]` (`D` is pydocstyle, `N` is pep8-naming), or flake8 and pylint config.
- Line length from the formatter config. Docstrings and comments follow it.
- Whether the code is type-hinted. That decides where types are written.

## Naming (PEP 8)

| Kind | Form |
| --- | --- |
| Module, package | `lower_snake`, short |
| Function, method, variable | `lower_snake` |
| Class, exception, type alias | `CapWords`; exceptions end in `Error` |
| Constant | `UPPER_SNAKE` at module level |
| Non-public | one leading underscore |
| Type variable | short `CapWords`: `T`, `KeyT` |

- Don't shadow builtins (`id`, `type`, `list`, `input`). Use a more specific name, or a trailing underscore if nothing better exists.
- Functions are verbs or verb phrases. Properties and values are nouns.
- Don't encode the type in the name (`user_list`, `name_str`). Name the content: `users`, `name`.

## Docstrings (PEP 257)

**What needs one**

- Every public module, class, function, and method.
- Not needed: private helpers whose name and signature say everything, overrides that add nothing to the parent's docstring, and tests with descriptive names. Follow the repo's lint rules if they are stricter.

**Form**

- Triple double quotes. The summary line starts right after the opening quotes, fits on one line, ends with a period, and is written in the imperative: "Return the invoice total.", not "Returns" or "This function returns".
- A blank line, then detail if it's needed. A one-line docstring is fine when the summary says everything.
- Don't repeat the signature in the summary.

**Content**

- With type hints, don't repeat types in the docstring. Describe meaning, units, allowed values, and what `None` or a default means.
- Without type hints, give the types in the docstring in the repo's style. Don't add type hints as part of a documentation task unless asked.
- Document every exception a caller is expected to handle, in `Raises`. Don't list exceptions that only signal a bug.
- Generators use `Yields` instead of `Returns`.
- State side effects: I/O, mutation of arguments, network calls, global state.
- Class docstrings describe what the class represents and list its public attributes. Constructor arguments are documented in the class docstring or in `__init__`, whichever the repo does; not both.
- Module docstrings say what the module provides, in a line or two. Skip an inventory of its contents.
- Use an `Examples` section when usage isn't obvious from the signature. Write it as a doctest if the repo runs doctests.

**Google style, the default**

```python
def fetch_invoices(customer_id: str, since: date | None = None) -> list[Invoice]:
    """Return the customer's invoices, newest first.

    Args:
        customer_id: The billing system's ID for the customer.
        since: Only include invoices issued on or after this date. If
            None, include all invoices.

    Returns:
        The matching invoices. Empty if the customer has none.

    Raises:
        CustomerNotFoundError: If no customer has this ID.
    """
```

## Comments

- Block comments sit above the code they describe, at the same indentation, as full sentences.
- Inline comments are rare, and separated from the code by two spaces.
- `# type: ignore`, `# noqa`, and `# pragma: no cover` name the specific code and give the reason: `# noqa: S608 - the table name comes from a constant`.

## Common findings

- The docstring lists arguments that no longer exist, or misses ones that were added.
- The docstring describes a return value or exception that the code doesn't produce.
- Types in the docstring contradict the type hints.
- The summary line restates the function name: `def get_user` with "Get user."
- Mutable or sentinel defaults whose meaning isn't explained.

## Checks after editing

Run what the repo has configured: `ruff check <paths>` and `ruff format --check <paths>`, or `pydocstyle`, `flake8`, `pylint`. Run doctests if the repo uses them and an example changed.
