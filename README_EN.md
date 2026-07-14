# Volc Skill

[简体中文](README.md) | **English**

A command-line Agent Skill for Volcengine Machine Learning Platform (veMLP), compatible with Codex and Claude Code. It helps agents manage development instances, submit jobs, and troubleshoot runtime status from a local shell or remote development machine.

## Features

- Configure and inspect the `volc` CLI
- List, start, stop, and reboot ML development instances
- Connect to remote development instances over SSH using connection details provided by the platform
- Submit custom training jobs with `volc ml_task submit`
- Submit Slurm jobs with `volc ml_task sbatch`
- Inspect task status, instances, logs, and container resource usage
- Generate, upload, and export task configs or code
- Locate official Volcengine Machine Learning Platform documentation

The Skill asks for confirmation before starting, stopping, or rebooting instances, cancelling tasks, or submitting long-running jobs. Access keys, resource queues, images, endpoints, and resource IDs are represented by placeholders; the repository contains no organization-specific information.

## Prerequisites

- A working `volc` CLI installation, verifiable with `volc version`
- Authentication configured through a local profile or environment variables
- Sufficient account permissions for the target development instances, resource queues, images, and tasks

Do not commit AK/SK credentials, SSH private keys, or configuration files containing credentials.

## Get the Source

```bash
git clone https://github.com/LioEinaudi/volc-skill.git
cd volc-skill
```

## Install in Codex

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R ./volc "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Restart Codex or open a new session. Invoke the Skill explicitly with `$volc`, or describe a Volcengine ML Platform task in natural language.

## Install in Claude Code

```bash
mkdir -p "$HOME/.claude/skills"
cp -R ./volc "$HOME/.claude/skills/"
```

Run `/skills` in Claude Code to confirm that `volc` was discovered, then invoke it with `/volc`. If `~/.claude/skills` did not exist before installation, restart Claude Code.

`volc/agents/openai.yaml` provides Codex UI metadata only. Claude Code ignores it without affecting `SKILL.md` or files under `references/`.

The core `SKILL.md` and `references/` layout follows the Agent Skills structure supported by both clients, so a separate Claude Code copy is unnecessary.

## Usage Examples

Codex:

```text
Use $volc to list the ML development instances available to the current account.
Use $volc to start a development instance and show me how to connect over SSH.
Use $volc to generate a job config from the current directory and submit a training job.
Use $volc to monitor a task's status and logs.
Use $volc to submit this sbatch script to a specified resource queue.
```

Claude Code:

```text
/volc List the ML development instances available to the current account.
/volc Start a development instance and show me how to connect over SSH.
/volc Generate a job config from the current directory, but do not submit it yet.
/volc Show the status and latest 500 log lines for the specified task.
```

Both clients can also invoke the Skill automatically from its `description`. The Skill checks the installed CLI with `volc --help` before applying the repository's command templates, reducing the risk of using flags from a different CLI version.

## **Important: Automated Task Submission**

> **The Skill gives an agent operational instructions; it is not an unattended scheduler by itself.** Live submission also requires a working `volc` CLI, non-interactive authentication, a complete task configuration, and permission to execute the command. Scheduled or CI submission additionally requires a scheduler, logging, and duplicate-prevention controls.

### 1. Prepare the Runtime

Inspect the CLI on the development machine, CI runner, or scheduling node:

```bash
command -v volc
volc version
volc ml_task --help
volc ml_task submit --help
```

Configure authentication through a platform-supported local profile, credential file, or CI secret. Credentials must be injected by the runtime; never put them in task YAML, scripts, Git, or agent conversations. Verify authentication and permissions with a read-only command:

```bash
volc ml_task list --limit 1
```

### 2. Prepare a Complete Task Configuration

Generate a template and fill every required field:

```bash
volc ml_task template --path job.yaml --example true
```

Review at least:

- A unique, traceable task name
- The container entrypoint and working directory
- The local code path and remote mount path
- The image URL and image permissions
- The resource queue, compute flavor, node count, and GPU count
- Environment variables, storage mounts, timeout, and retry policy

Do not ask the agent to guess queues, images, regions, instance IDs, or credentials. Shared configurations should be secret-free templates with environment-specific values injected at deployment time.

### 3. Allow the Agent to Execute

Installing the Skill does not grant shell permissions. Configure the client separately to allow `volc ml_task`, while retaining human confirmation for high-risk operations such as cancelling tasks, deleting resources, or stopping development instances.

For a one-time submission, explicitly request live execution and provide the configuration:

```text
Use $volc to inspect ./job.yaml. If all fields are complete and the account, queue, and image are available, submit it now. I confirm that this task will consume billable compute resources. Return the task ID and check its initial status.
```

Use `/volc` instead of `$volc` in Claude Code. If the request only asks how to submit or still contains placeholders, the Skill should show the command without executing it.

### 4. Run a Preflight Before Every Submission

```bash
set -euo pipefail
test -r ./job.yaml
command -v volc >/dev/null
volc version
volc ml_task list --limit 1 >/dev/null
volc ml_task submit --help >/dev/null
```

A human or project validator should also check the YAML entrypoint, code directory, image, queue, and compute flavor. Submit only after every check succeeds:

```bash
volc ml_task submit --conf ./job.yaml
```

Preserve standard output, standard error, and the exit code. Record the task ID after success and monitor it with:

```bash
volc ml_task get --id <task-id>
volc ml_task instance list --id <task-id>
volc ml_task logs --task <task-id> --lines 200
```

### 5. Wrap Non-Interactive Submission

Create a minimal wrapper in a private deployment repository. Do not put credentials in the script:

```bash
#!/usr/bin/env bash
set -euo pipefail

conf=${1:?"usage: submit-volc-task <job.yaml>"}
test -r "$conf"
command -v volc >/dev/null

volc version
volc ml_task list --limit 1 >/dev/null
volc ml_task submit --help >/dev/null

started_at=$(date -u +%Y-%m-%dT%H:%M:%SZ)
printf 'submit_started_at=%s config=%s\n' "$started_at" "$conf"
volc ml_task submit --conf "$conf"
```

The exit code should directly represent submission success. Do not infer success from a log keyword, and do not retry indefinitely: a network timeout can occur after the service has created the task.

### 6. Connect cron or CI

Run the same script manually before enabling a scheduler. Example cron entry:

```cron
15 2 * * * flock -n "$HOME/.cache/volc-submit.lock" "$HOME/bin/submit-volc-task" "$HOME/jobs/job.yaml" >> "$HOME/volc-submit.log" 2>&1
```

In CI, separate configuration validation, preflight, and submission. Protect submission with an environment gate, branch restriction, or manual approval. Every scheduler should provide:

- A single-instance lock to prevent concurrent submissions
- A unique task name or external idempotency record to prevent duplicates
- Explicit timeouts and bounded retries
- Persistent logs, task IDs, and submission timestamps
- Failure notifications and a human cancellation path

### 7. Validate the Automation Path

Use a short, low-cost task for the first end-to-end test:

1. The scheduler can find both `volc` and the task configuration.
2. Non-interactive authentication works with only the required permissions.
3. A failed preflight prevents submission.
4. A successful submission records the task ID, status, and logs.
5. Repeated triggers do not create concurrent or duplicate tasks.
6. Credentials do not appear in logs, configurations, or agent output.

Enable long-running production training only after these checks pass.

## Update

Pull the latest repository changes and copy the Skill again:

```bash
git pull --ff-only
cp -R ./volc "${CODEX_HOME:-$HOME/.codex}/skills/"
cp -R ./volc "$HOME/.claude/skills/"
```

## Repository Layout

```text
volc/
├── SKILL.md                         # Skill entry point and operational guardrails
├── agents/openai.yaml               # Codex UI metadata; ignored by Claude Code
└── references/
    ├── cli-commands.md              # Generic CLI commands and job templates
    └── ml-platform-docs.md          # Official documentation index
```

## Local Packaging

The repository tracks only Skill source files. To transfer the Skill offline, create an archive from the repository root:

```bash
tar -czf volc-skill.tar.gz volc
```

The generated archive is excluded by `.gitignore` and should not be committed to the source repository. GitHub also provides source archives for every commit and release tag.

## Sources

- [Volcengine Machine Learning Platform documentation](https://www.volcengine.com/docs/6459)
- [Volcengine CLI usage documentation](https://www.volcengine.com/docs/6459/72394)
- [Claude Code Skills documentation](https://code.claude.com/docs/en/skills)
