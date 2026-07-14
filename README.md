# Volc Skill

**简体中文** | [English](README_EN.md)

适用于 Codex 和 Claude Code 的火山引擎机器学习平台（veMLP）命令行 Agent Skill，帮助代理在本地终端或远程开发机中完成开发机管理、任务提交与运行状态排查。

## 功能范围

- 配置和检查 `volc` CLI
- 查询、启动、停止和重启 ML 开发机
- 按平台提供的连接信息通过 SSH 使用远程开发机
- 使用 `volc ml_task submit` 提交自定义训练任务
- 使用 `volc ml_task sbatch` 提交 Slurm 任务
- 查看任务状态、实例、日志和容器负载
- 生成、上传和导出任务配置或代码
- 查找火山引擎机器学习平台官方文档

涉及启动、停止、重启、取消任务或提交长时间运行任务时，Skill 会先要求确认。示例中的访问密钥、资源队列、镜像、地址和资源 ID 均使用占位符，不包含组织内部信息。

## 使用前提

- 已安装可用的 `volc` CLI，并可通过 `volc version` 检查版本
- 已使用本地配置或环境变量完成身份认证
- 当前账号具有目标开发机、资源队列、镜像和任务的相应权限

不要把 AK/SK、SSH 私钥或包含凭据的配置文件提交到仓库。

## 获取源码

```bash
git clone https://github.com/LioEinaudi/volc-skill.git
cd volc-skill
```

## 安装到 Codex

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R ./volc "${CODEX_HOME:-$HOME/.codex}/skills/"
```

重新启动 Codex 或开启新会话，然后通过 `$volc` 显式调用，也可以直接用自然语言描述火山引擎 ML Platform 任务。

## 安装到 Claude Code

```bash
mkdir -p "$HOME/.claude/skills"
cp -R ./volc "$HOME/.claude/skills/"
```

在 Claude Code 中运行 `/skills` 确认 `volc` 已被发现，然后通过 `/volc` 调用。若安装前 `~/.claude/skills` 不存在，需要重新启动 Claude Code。

`volc/agents/openai.yaml` 仅提供 Codex 界面元数据；Claude Code 会忽略该文件，不影响 `SKILL.md` 和 `references/` 的加载。

核心 `SKILL.md` 与 `references/` 采用两个客户端均支持的 Agent Skills 目录结构，因此无需维护 Claude Code 专用副本。

## 使用示例

Codex：

```text
使用 $volc 列出当前账号下的 ML 开发机。
使用 $volc 启动指定开发机，并告诉我如何通过 SSH 连接。
使用 $volc 根据当前目录生成任务配置并提交训练任务。
使用 $volc 跟踪某个任务的状态和日志。
使用 $volc 把这个 sbatch 脚本提交到指定资源队列。
```

Claude Code：

```text
/volc 列出当前账号下的 ML 开发机。
/volc 启动指定开发机，并告诉我如何通过 SSH 连接。
/volc 根据当前目录生成任务配置，但先不要提交。
/volc 跟踪指定任务的状态和最近 500 行日志。
```

两个客户端都可以根据 `description` 自动触发该 Skill。Skill 会优先检查本机安装版本的 `volc --help`，再使用仓库中的命令模板，避免因 CLI 版本差异直接执行过期参数。

## **重点：自动化提交任务**

> **Skill 只向 Agent 提供操作流程和命令知识，不会单独完成无人值守调度。** 要让 Agent 实际提交任务，运行环境还必须具备可用的 `volc` CLI、非交互身份认证、完整任务配置和执行命令的权限。定时或 CI 提交还需要调度器、日志和防重复机制。

### 1. 准备运行环境

在执行任务的开发机、CI Runner 或调度节点上检查 CLI：

```bash
command -v volc
volc version
volc ml_task --help
volc ml_task submit --help
```

使用平台支持的本地配置、凭据文件或 CI Secret 完成身份认证。凭据必须由运行环境注入，不要写入任务 YAML、提交脚本、Git 仓库或 Agent 对话。先用只读命令确认认证和账号权限：

```bash
volc ml_task list --limit 1
```

### 2. 准备完整任务配置

先生成模板，再填写所有必需字段：

```bash
volc ml_task template --path job.yaml --example true
```

提交前至少核对：

- 唯一且可追踪的任务名称
- 容器入口命令和容器内工作目录
- 本地代码路径及远端挂载路径
- 镜像地址和镜像访问权限
- 资源队列、计算规格、节点数和 GPU 数
- 环境变量、存储挂载、超时和重试策略

不要让 Agent 猜测资源队列、镜像、区域、实例 ID 或凭据。团队配置应使用无敏感信息的模板，并由部署环境注入差异化参数。

### 3. 授予 Agent 执行权限

安装 Skill 不会自动授予 Shell 权限。需要在所用客户端中单独允许执行 `volc ml_task`，同时保留对取消任务、删除资源、停止开发机等高风险命令的人工确认。

一次性提交时，应明确要求现场执行并提供配置文件：

```text
使用 $volc 检查 ./job.yaml；字段完整且账号、队列和镜像均可用后，实际执行提交。我确认本次任务会产生计算资源费用。提交后返回任务 ID，并检查首次状态。
```

Claude Code 中将 `$volc` 改为 `/volc`。如果只说“告诉我怎么提交”或配置仍有占位符，Skill 应只给出命令而不执行。

### 4. 每次提交前执行 preflight

```bash
set -euo pipefail
test -r ./job.yaml
command -v volc >/dev/null
volc version
volc ml_task list --limit 1 >/dev/null
volc ml_task submit --help >/dev/null
```

还应由人工或项目脚本验证 YAML 中的入口、代码目录、镜像、队列和计算规格。全部检查通过后执行：

```bash
volc ml_task submit --conf ./job.yaml
```

保存标准输出、标准错误和退出码。提交成功后记录任务 ID 并监控：

```bash
volc ml_task get --id <task-id>
volc ml_task instance list --id <task-id>
volc ml_task logs --task <task-id> --lines 200
```

### 5. 封装非交互提交脚本

可在私有部署仓库中创建最小包装脚本；不要把凭据写入脚本：

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

脚本退出码应直接反映提交是否成功。不要只凭日志关键字判定成功，也不要无限自动重试，因为网络超时可能发生在服务端已经创建任务之后。

### 6. 接入 cron 或 CI

先手动运行同一脚本并确认提交成功。cron 示例：

```cron
15 2 * * * flock -n "$HOME/.cache/volc-submit.lock" "$HOME/bin/submit-volc-task" "$HOME/jobs/job.yaml" >> "$HOME/volc-submit.log" 2>&1
```

在 CI 中将配置校验、preflight 和提交拆成独立步骤，并为提交步骤设置受保护环境、分支限制或人工审批。无论使用哪种调度器，都应配置：

- 单实例锁，防止同一周期并发提交
- 唯一任务名或外部幂等记录，防止重跑产生重复任务
- 明确的超时和有限重试
- 持久化日志、任务 ID 和提交时间
- 失败通知以及人工取消入口

### 7. 验收自动化链路

先使用低成本短任务完成端到端验证：

1. 调度器能够找到 `volc` 和任务配置。
2. 非交互认证有效，且只拥有所需权限。
3. preflight 失败时不会进入提交步骤。
4. 提交成功后能记录任务 ID、状态和日志。
5. 重复触发不会并发或重复创建同一任务。
6. 日志、配置和 Agent 输出中均不包含凭据。

完成以上验收后，再启用正式的长时间训练任务。

## 更新

在仓库目录中拉取最新版本，再重新复制 Skill：

```bash
git pull --ff-only
cp -R ./volc "${CODEX_HOME:-$HOME/.codex}/skills/"
cp -R ./volc "$HOME/.claude/skills/"
```

## 仓库结构

```text
volc/
├── SKILL.md                         # Skill 入口与操作原则
├── agents/openai.yaml               # Codex 界面元数据，Claude Code 忽略
└── references/
    ├── cli-commands.md              # 通用 CLI 命令与任务模板
    └── ml-platform-docs.md          # 官方文档索引
```

## 本地打包

仓库只跟踪 Skill 源文件。需要离线传输时，可在仓库根目录本地生成归档包：

```bash
tar -czf volc-skill.tar.gz volc
```

生成的归档包已被 `.gitignore` 排除，不应提交到源码仓库。GitHub 本身也会为每个提交和版本标签自动提供源码归档。

## 资料来源

- [火山引擎机器学习平台文档](https://www.volcengine.com/docs/6459)
- [命令行工具使用文档](https://www.volcengine.com/docs/6459/72394)
- [Claude Code Skills 文档](https://code.claude.com/docs/en/skills)
