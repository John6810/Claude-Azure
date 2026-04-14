# Generate Terraform Tests

Generate `.tftest.hcl` files for module `$ARGUMENTS`. Use `all` for bulk generation.

Invoke `test-creator` agent which analyzes module source and generates:
- `naming.tftest.hcl` — computed name logic
- `tags.tftest.hcl` — tag merge + CreatedOn
- `constraints.tftest.hcl` — KV 24 chars, ACR no hyphens (if applicable)
- `integration.tftest.hcl` — real provider (free modules only)
