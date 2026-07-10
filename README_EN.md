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
