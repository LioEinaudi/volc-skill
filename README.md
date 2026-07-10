# Volc Skill

用于 Codex 的火山引擎机器学习平台（veMLP）命令行 Skill，帮助代理在终端或远程开发机中完成开发机管理、任务提交与运行状态排查。

## 功能范围

- 配置和检查 `volc` CLI
- 查询、启动、停止和重启 ML 开发机
- 按平台提供的连接信息通过 SSH 使用远程开发机
- 使用 `volc ml_task submit` 提交自定义训练任务
- 使用 `volc ml_task sbatch` 提交 Slurm 任务
- 查看任务状态、实例、日志和容器负载
- 生成、上传和导出任务配置或代码
- 查找火山引擎机器学习平台官方文档

涉及停止、重启、取消任务或提交长时间运行任务时，Skill 会先要求确认。示例中的访问密钥、资源队列、镜像和资源 ID 均使用占位符，不包含组织内部信息。

## 安装

```bash
git clone https://github.com/LioEinaudi/volc-skill.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R ./volc-skill/volc "${CODEX_HOME:-$HOME/.codex}/skills/"
```

重新启动 Codex 或开启新会话后即可使用。

## 使用示例

```text
使用 $volc 列出当前账号下的 ML 开发机。
使用 $volc 启动指定开发机，并告诉我如何通过 SSH 连接。
使用 $volc 根据当前目录生成任务配置并提交训练任务。
使用 $volc 跟踪某个任务的状态和日志。
使用 $volc 把这个 sbatch 脚本提交到指定资源队列。
```

Skill 会优先检查本机安装版本的 `volc --help`，再使用仓库中的命令模板，避免因 CLI 版本差异直接执行过期参数。

## 仓库结构

```text
volc/
├── SKILL.md                         # Skill 入口与操作原则
├── agents/openai.yaml               # Codex 界面元数据
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
