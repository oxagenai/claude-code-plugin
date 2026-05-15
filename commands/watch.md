---
description: Start the Oxagen monitor supervisor (Linear backlog watcher in Phase 1).
argument-hint: "[--daemons linear,feedback,logs,research] [--interval-ms N]"
allowed-tools: Bash
---

# /oxagen:watch — start monitor daemons

Run the supervisor so the backlog gets worked even when the user is
away from the keyboard.

## Steps

1. Verify `.oxagen.toml` exists.
2. Run:

```bash
oxagen watch $ARGS
```

3. The process is long-running. Stream its log output back to the user
   and remind them the daemons stop when the laptop sleeps. To go
   always-on, deploy the Cloud Run module under
   `infra/terraform/cloud-run-oxagen-cli-daemons.tf`.
