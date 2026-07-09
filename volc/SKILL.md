---
name: volc
description: "Use when Codex needs to operate or explain 火山引擎/Volcengine Machine Learning Platform from a shell or remote development machine: configuring the volc CLI, listing/starting/stopping/rebooting ML development instances, SSH-oriented devinstance workflows, submitting custom training jobs or Slurm sbatch jobs with volc ml_task, monitoring task status/logs/instances/top, exporting/uploading task code or configs, choosing resource queues/images, or citing official Volcengine ML Platform docs."
---

# Volc

## Workflow

Use this skill for 火山引擎机器学习平台 (ML Platform / veMLP) command-line work.

1. Confirm the local CLI surface before issuing commands:
   - `volc version`
   - `volc configure --help`
   - `volc ml_devinstance --help`
   - `volc ml_task --help`
2. For exact command flags and templates, read [references/cli-commands.md](references/cli-commands.md).
3. For official documentation links, doc IDs, and how to refresh the doc tree, read [references/ml-platform-docs.md](references/ml-platform-docs.md).
4. Prefer non-destructive inspection first: list/get/status/log commands before start/stop/reboot/cancel.
5. Do not print or request AK/SK secrets in chat. Use placeholders such as `$VOLC_AK`, `$VOLC_SK`, `$VOLC_REGION`, and `$VOLC_PROFILE`.

## Common Paths

For "在远程开发机起任务":

1. Ensure the development instance is running with `volc ml_devinstance list` and `volc ml_devinstance start --id <devinstance-id>` if needed.
2. Connect through the SSH method shown by the platform or `volc ml_devinstance get --id <devinstance-id>` if it exposes connection details.
3. From the dev machine workspace, submit a job with either:
   - `volc ml_task submit` for custom jobs.
   - `volc ml_task sbatch` for Slurm scripts.
4. Monitor with `volc ml_task list`, `volc ml_task get`, `volc ml_task logs -f`, `volc ml_task instance list`, and `volc ml_task top`.

## Guardrails

- Confirm before destructive or cost-impacting commands: `stop`, `reboot`, `cancel`, deleting resources, or long-running job submissions.
- Prefer `--resource_queue_name` over deprecated `--resource_group_id`; use `--resource_queue_id` only when a queue name is unavailable.
- For reproducible job submissions, start from `volc ml_task template --path <file> --example true`, then override with flags or `--set`.
- Remember `--user_code_path/--cp` uploads files to TOS; `.gitignore` under that path affects uploaded code.
- For symlinks, choose explicitly: `--copy-links` copies referents, while `--links` recreates symlinks.
