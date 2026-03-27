# Infra Mode — continuous-claude rules for infrastructure projects

## You are in INFRA MODE

This project contains infrastructure code (Terraform, Azure CLI, AWS CLI, etc.)
alongside application code. When running under continuous-claude, you MUST follow
these rules to avoid crashing the session.

## The One Rule

**Only make code changes. Never deploy.**

Each continuous-claude iteration must produce a small, self-contained code change
that can be committed and PR'd in under 5 minutes. If a task requires running
live infrastructure, skip it and note it for manual execution.

## Forbidden Commands

Do NOT run any of these in a continuous-claude iteration:

- `terraform apply`, `terraform destroy`, `terraform plan` (reading `.tf` files is fine)
- `az vm create`, `az group create`, `az vm list -d`, or any `az` command that provisions/queries live resources
- `aws ec2 run-instances`, `aws cloudformation deploy`, or any provisioning `aws` command
- `ansible-playbook`, `ansible` (reading playbooks is fine)
- `docker-compose up`, `docker build` (reading Dockerfiles is fine)
- `kubectl apply`, `helm install`
- Any SSH command (`ssh`, `scp`, `sftp`)
- Any command with `sleep` > 5 seconds

**Read-only inspection is OK**: `terraform fmt`, `terraform validate`, `az --version`,
reading state files, parsing JSON outputs, etc.

## Cleanup — Never Leave Things Running

Before finishing an iteration, verify you have not left anything running:
- No background processes (`&`, `nohup`, tasks)
- No open SSH tunnels or port forwards
- No running containers you started
- No terraform locks held

If something was already running when you started, leave it alone — just don't
start new long-running processes.

## Micro-PR Strategy

Each iteration = ONE of these:
- Fix a bug in a script (e.g., wrong variable reference, path issue)
- Add/update a function in a library file
- Fix a template or config file
- Add error handling or validation
- Update documentation or comments
- Refactor a section for clarity

Each iteration != any of these:
- Deploy + verify + fix + redeploy
- Run a full E2E test pipeline
- Wait for infrastructure to come up
- Debug a live environment

## TODO.md Format

The project's TODO.md should have two sections:

```
## Code Tasks
- [ ] Fix $AZ variable in gather_report_data.sh
- [ ] Fix config path resolution in 2_status.sh
- [ ] Add timeout to az vm list calls

## Infra Tasks (manual — do not pick these)
- [ ] Deploy fresh E2E environment and verify report
- [ ] Run full status check against live VMs
- [ ] Generate PDF report with live screenshots
```

**Only pick tasks from "## Code Tasks".**

If you discover a code bug while reading, add it to Code Tasks and fix it in
the current iteration. If verifying the fix requires live infra, add a
corresponding Infra Tasks item.

## When You Find Nothing To Do

If all Code Tasks are checked off but Infra Tasks remain:
1. Review the Infra Tasks — can any be decomposed into code-only prep work?
2. If yes, add new Code Tasks and work on those
3. If no, output CONTINUOUS_CLAUDE_PROJECT_COMPLETE

## Error Budget

- If a command takes more than 30 seconds, it's probably wrong for this mode
- If you're about to `sleep`, you're probably doing infra work — stop
- If you need to "wait for" something, it's an infra task — skip it
