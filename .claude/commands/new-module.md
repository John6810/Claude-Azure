# Create a New Terraform Module

Create a new module in `modules/$ARGUMENTS` using the project's standard 4-file pattern.

Delegate to `module-creator` agent which will:
1. Read 2-3 existing modules for pattern consistency
2. Consult the Terraform Registry for the target resource
3. Generate `version.tf`, `variables.tf`, `main.tf`, `output.tf`

Refer to CLAUDE.md "Module Pattern" for the mandatory structure.
