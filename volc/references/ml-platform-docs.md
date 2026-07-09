# Official Volcengine ML Platform Docs

Public documentation index for 火山引擎机器学习平台 (ML Platform / veMLP). Use these links and IDs to find the current official page, then verify exact behavior against the installed `volc` CLI.

## Primary Sources

- Product docs library: `6459` - 机器学习平台 / ML Platform.
- Product docs page: `https://www.volcengine.com/docs/6459`
- Command-line usage page: `https://www.volcengine.com/docs/6459/72394`

## Key Doc IDs

Use `https://www.volcengine.com/docs/6459/<doc-id>` for each page.

Command-line tools:

- `80260` - 命令行工具.
- `72394` - 使用文档.
- `114719` - 升级指南.
- `80261` - 变更记录.

Development machines:

- `127869` - 开发机.
- `129233` - 创建开发机.
- `1283201` - 开发机生命周期管理.
- `129240` - 通过 SSH 远程连接开发机.
- `129246` - 关闭开发机.
- `129249` - 重启开发机.
- `129250` - 删除开发机.
- `129252` - 使用 WebIDE 开发代码.
- `1177900` - 在开发机中创建 Docker 容器.
- `1526760` - 更换开发机镜像.
- `196532` - 设置开发机自动关机规则.
- `1874762` - 开发机上使用 RDP GUI 实践.
- `2344354` - 在开发机上用 MobilityGen 生成导航数据.

Custom tasks:

- `72307` - 自定义任务.
- `72350` - 创建单机 / 分布式训练任务.
- `1175114` - 优先级调度策略.
- `72351` - 查看 TensorBoard 日志.
- `72349` - 查看任务的状态 / 监控 / 日志.
- `147996` - 预付费场景下的闲时任务.
- `75221` - 发起 TensorFlowPS 分布式训练.
- `75222` - 发起 PyTorchDDP 分布式训练.
- `75223` - 发起 MPI 分布式训练.
- `75224` - 发起 BytePS 分布式训练.
- `1262811` - 使用 RAY 计算引擎提交分布式任务.
- `96563` - 通过 RDMA 网络加速训练.
- `119595` - 验证镜像是否支持 RDMA.
- `75329` - 通用环境变量列表.
- `1348196` - 疑似故障节点上报.
- `1453766` - 历史任务归档功能说明.
- `1541502` - Py-spy 采集分析.
- `1403170` - CPU 性能分析.
- `1738901` - GPU 性能分析.
- `2139861` - Hang 检测能力.
- `2275080` - 常态化持续火焰图.

Resource groups and queues:

- `72304` - 资源组.
- `1254768` - 创建资源组.
- `111937` - 创建资源组.
- `111939` - 为资源组续费 / 更配 / 退订.
- `111941` - 创建资源队列.
- `111942` - 管理队列内的用户.
- `111944` - 为队列更配 / 转让资源.
- `1947175` - 设置低利用率自定义任务关闭规则.
- `1339028` - 负载排队中状态常见原因说明.
- `1449547` - GPU/CPU 灵活配比使用指南.

Images:

- `72305` - 镜像仓库.
- `72341` - 预置镜像列表.
- `72342` - 构建自定义镜像.
- `106867` - 迁移外部镜像到镜像仓库.

Slurm and best practices:

- `107346` - 通过自定义镜像提交 Slurm 任务.
- `108993` - 多实例命令批量执行工具.
- `1109790` - 通过工作流串联训练与评测任务.
- `1415110` - 如何在容器内查看进程信息.

API reference:

- `1541519` - API 参考.
- `1544742` - 自定义任务.
- `1544751` - CreateJob - 创建自定义任务.
- `1544749` - GetJob - 查询自定义任务.
- `1544747` - ListJobs - 查询自定义任务列表.
- `1544748` - ListJobInstances - 查询自定义任务实例列表.
- `1544744` - StopJob - 停止自定义任务.
- `1544752` - 开发机.
- `1544761` - CreateDevInstance - 创建开发机.
- `1544759` - GetDevInstance - 获取开发机.
- `1544758` - ListDevInstances - 开发机列表.
- `1544756` - StartDevInstance - 启动开发机.
- `1544755` - StopDevInstance - 停止开发机.
- `1544757` - RebootDevInstance - 重启开发机.
- `1544760` - DeleteDevInstance - 删除开发机.
- `1544762` - CancelIdleShutdown - 取消开发机闲时关机.
- `1544725` - 队列.
- `1544729` - ListResourceQueues - 资源队列列表.
- `1544730` - GetResourceQueue - 查询资源队列.

## Search Heuristics

When asked for a workflow:

- Start with CLI help for the installed version because `volc` behavior can drift from docs.
- Use docs ID `72394` for CLI syntax and docs ID `129240` for SSH details.
- Use docs ID `72350` for creating training jobs and `72349` for state/log/monitoring operations.
- Use docs ID `107346` when the user asks about Slurm or custom images.
- Use docs ID `111941` or API docs under `1544725` when the blocker is queue selection.
