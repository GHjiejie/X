# 平台能力文档目录

| 属性 | 内容 |
|---|---|
| 文档 ID | DOC-CATALOG |
| 版本 | v0.1 |
| 日期 | 2026-09-13 |
| 状态 | 模块拆分初稿；不代表生产批准 |
| 上一版本 | 无；首次建立能力目录 |
| 依据 | 用户提供的八类能力表；总蓝图 v0.2 |
| 本次变更 | 新建八个模块目录、版本规范与总蓝图快照，保留原始文档 |

阅读入口：[版本规范 v0.1](versioning.v0.1.md)、[总蓝图 v0.2 快照](00-architecture/financial-agent-platform-blueprint.v0.2.md)、[原始蓝图（保留）](financial-agent-platform-blueprint.md)。

## 本次固定的文档组合

| 组成 | 目录 | 本次版本 | 能力范围 |
|---|---|---|---|
| 员工工作台 | `01-employee-workbench` | [设计 v0.1](01-employee-workbench/design.v0.1.md) | 应用目录、对话、附件、进度、引用、草稿、审阅、导出、反馈 |
| 平台管理 | `02-platform-management` | [设计 v0.1](02-platform-management/design.v0.1.md) | 项目与角色、模板与版本、目录、配额、发布、评测与审计入口 |
| Run 服务 | `03-run-service` | [设计 v0.1](03-run-service/design.v0.1.md) | 任务创建、排队、状态、事件流、取消、恢复、定时入口 |
| Agent Runtime | `04-agent-runtime` | [设计 v0.1](04-agent-runtime/design.v0.1.md) | Deep Agents、执行图、子任务、持久状态、重试、人工中断 |
| 知识服务 | `05-knowledge-service` | [设计 v0.1](05-knowledge-service/design.v0.1.md) | 内容接入、ACL 同步、时间过滤、混合检索、重排与证据 |
| 工具网关 | `06-tool-gateway` | [设计 v0.1](06-tool-gateway/design.v0.1.md) | 工具注册、参数校验、身份委托、限流、沙箱与执行确认 |
| 模型网关 | `07-model-gateway` | [设计 v0.1](07-model-gateway/design.v0.1.md) | 统一接口、白名单、密级路由、容量、成本与内容治理 |
| 观测与审计 | `08-observability-audit` | [设计 v0.1](08-observability-audit/design.v0.1.md) | LangSmith Cloud 调试与评测、独立审计证据 |

## 共用边界与阅读顺序

先阅读总蓝图第 0 节 P0 门禁，再阅读目标模块及其依赖。总蓝图第 13 节对前文的状态、数据处理与排期覆盖规则继续有效。模块文档是现有设计的职责拆分，不是新的生产准入决定；本次未确认的配置、配额、SLO 和审批结论保持待定。

- Deep Agents 为首选执行框架，Run 服务负责对外任务生命周期；Runtime 负责图内执行，二者不能争夺同一状态的写入权。
- OSS 承担原文、附件、报告等大对象；数据库承担版本、权限、任务、审批等需要事务的状态。
- 三方模型经模型网关接入。模型、OCR、Embedding、重排与评测各路径均受部署和数据门禁约束。
- 不建设自有 Agent 可观测平台；LangSmith Cloud 仅在允许的出网范围内启用。金融审计与调试 Trace 分开。
- 首期先验证单域、单数据源、单模板；P0 未通过前仅使用合成或获准低敏数据。

## 跨模块交接

| 链路 | 主责 | 协作边界 |
|---|---|---|
| 用户创建任务 | 工作台 → Run 服务 | 工作台提交输入；服务端鉴权并分配任务与版本 |
| 发布 Agent | 平台管理 → Run / Runtime | 只交付不可变发布配置，旧运行保留固定版本 |
| 执行与恢复 | Run 服务 → Runtime | Run 持有外部状态；Checkpoint 保存图内恢复点 |
| 证据与计算 | Runtime → 知识 / 工具网关 | 检索权限、计算口径和业务动作由对应服务强制执行 |
| 模型调用 | Runtime / 知识 / 评测 → 模型网关 | 统一控制各类模型的数据路由与预算 |
| 调试与取证 | 各模块 → 观测与审计 | 脱敏 Trace 与持久审计采用不同访问和保留策略 |

未来修改某一模块时新增其版本文件，并新增目录索引版本。旧目录索引始终指向当时的文档组合，不跟随新版本改变。
