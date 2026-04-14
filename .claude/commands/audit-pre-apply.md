# Pre-Apply Audit

Audit `$ARGUMENTS` before `terragrunt apply`.

Invoke `deployment-reviewer` agent on the target path. It checks all 10 points from the project's gotcha list (mock_outputs, SubnetWithNsg keys, KV 24 chars, Palo routes, cross-sub deps, etc.).
