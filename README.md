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
