# Terraform

Applies to OpenTofu as well.

## Detect the conventions

- `.tflint.hcl`: rules such as `terraform_documented_variables`, `terraform_documented_outputs`, and `terraform_naming_convention` show what is enforced.
- `.terraform-docs.yml`, or `<!-- BEGIN_TF_DOCS -->` markers in a README: the descriptions feed a generated table, so they must read well as a table cell.
- Existing `variables.tf` and `outputs.tf`: sentence style, trailing periods, and how defaults and units are described.

## Naming

- `lower_snake_case` for every name: resources, data sources, variables, outputs, locals, modules. No hyphens.
- Don't repeat the resource type in the resource name: `aws_s3_bucket.logs`, not `aws_s3_bucket.logs_bucket`.
- Name a resource for its role. Use `this` or `main` only when the module has exactly one resource of that type and no better name exists; follow whichever the repo uses.
- Singular nouns for single values, plural for lists and maps: `subnet_ids`, `tags`.
- Booleans start with `enable_`, `create_`, `is_`, or `has_`, and are positive: `enable_encryption`, not `disable_encryption`.
- Put units in the names of numeric variables: `retention_days`, `timeout_seconds`, `disk_size_gb`.
- Outputs follow `<thing>_<attribute>`: `bucket_arn`, `vpc_id`.

**Renames change infrastructure or interfaces. Report them; don't apply them unless asked.**

- A resource or module rename needs a `moved` block, or Terraform plans to destroy and recreate it.
- A variable or output rename breaks every caller of the module.

## Descriptions

Every `variable` and `output` has a `description`.

**Variables**

- Say what the value controls and what effect it has, not what the name already says. `"The name"` for `var.name` is a finding.
- Include what the type can't express: units, format (`"CIDR block, such as 10.0.0.0/16"`), allowed values, and what `null` or an empty value means.
- Don't state the default value. It is in `default` and generated docs show it. Do explain the default's effect when it isn't obvious.
- Start with a capital letter and write a sentence or noun phrase. Follow the repo on trailing periods.
- Keep it to one or two sentences. Generated tables put it in one cell, so avoid newlines and lists. For a complex `object` type, describe each attribute in a heredoc only if the repo already does so.
- Mark `sensitive = true` where it applies, and don't put example secrets in descriptions.

```hcl
variable "retention_days" {
  description = "How long to keep log events, in days. Must be a value CloudWatch accepts, such as 30, 90, or 365."
  type        = number
  default     = 30
}
```

**Validation**

- `error_message` tells the user what is allowed, as a full sentence ending in a period: `"The environment must be one of: dev, staging, prod."`

**Outputs**

- Say what the value is and, where it helps, what it's used for: `"ARN of the log bucket, for use in IAM policies."`

## Comments

- Use `#`. Not `//`, and `/* */` only where the repo already does.
- Explain what a reader can't see from the block:
  - `lifecycle` settings (`ignore_changes`, `prevent_destroy`, `create_before_destroy`) and why they are there;
  - explicit `depends_on`;
  - non-trivial `count` and `for_each` expressions, and complex `locals`;
  - hard-coded IDs, ARNs, CIDRs, and magic numbers;
  - provider or version pins that work around a bug, with a link to the issue.
- Don't comment what a resource is. The type and name say that.
- A file doesn't need a header comment if its name says what it holds (`variables.tf`, `iam.tf`).

## Checks after editing

`terraform fmt -check -recursive`, then `terraform validate` if the directory is initialised, and `tflint` if configured. If the README has a generated `terraform-docs` block, tell the user it needs regenerating; don't edit the block by hand.
