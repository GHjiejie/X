# 执行平面详细设计

| 属性 | 内容 |
|---|---|
| 文档 ID | ARCH-EXECUTION-PLANE |
| 版本 | v0.1 |
| 日期 | 2026-09-13 |
| 状态 | 分区详细设计初稿；继承总蓝图 P0 未通过状态 |
| 上一版本 | [总蓝图 v0.2](../financial-agent-platform-blueprint.v0.2.md) |
| 依据 | [能力目录 v0.1](../../catalog.v0.1.md)、总蓝图 v0.2、平台能力表 |
| 责任团队 | 运行平台与 Agent 工程团队 |
| 本次变更 | 首次从总体结构图拆出执行平面；不覆盖既有文档 |

## 1. 目标与边界

执行平面接收控制平面已发布的版本，运行 Deep Agents 和外层 LangGraph 治理图，协调知识、模型、工具和人工审阅。它拥有 Run 生命周期的执行协调权，但不拥有业务数据库通用写权限，也不自行决定数据访问范围。

## 2. 结构设计

~~~mermaid
flowchart TB
    API[Run API] --> Q[持久任务与租约]
    Q --> W[Deep Agents Worker]
    W --> G[外层 LangGraph 治理节点]
    G --> K[知识服务]
    G --> M[模型网关]
    G --> T[工具网关]
    G <--> H[人工审阅]
    G --> C[Checkpoint / Store]
    G --> L[LangSmith Cloud Trace]
    Q --> A[独立审计事件]
~~~

Run 表是对外状态权威，Checkpoint 只保存图内恢复状态，Approval 保存人工决定，Outbox 保存待发送意图，ToolExecution 保存下游回执。各对象通过引用和状态不变量关联，不能假设跨存储操作天然原子。

## 3. 组件职责

| 组件 | 详细职责 | 明确不负责 |
|---|---|---|
| Run API | 创建、查询、事件流、取消、恢复、审阅和导出；服务端构造签名 Run context | 不接受客户端覆盖 tenant、scope 或版本 |
| Task Coordinator | 任务表、队列、租约、重试和 worker epoch；确保单一有效写入者 | 不把至少一次投递伪装为 exactly-once |
| Deep Agent Worker | 使用 create_deep_agent 做规划、文件/上下文管理和受控子 Agent | 不访问宿主文件、长期凭据、任意 Shell 或任意 URL |
| Governance Graph | 用 LangGraph 固定身份、ACL、预算、审批、恢复、结束条件和副作用前检查 | 不让模型选择或降低风险等级 |
| Human Review Bridge | 暂停并展示 interrupt、证据、参数哈希、版本和期限，审批后用原 thread 恢复 | 不允许申请人批准自己的动作 |
| Execution Adapters | 调用知识、模型和工具网关，转换版本化请求和结果 | 不直接访问业务数据库或供应商密钥 |
| State Writer | 协调 Run、Checkpoint、Approval、Outbox、ToolExecution 和审计关联 | 不吞掉 UNKNOWN、租约冲突或序号缺口 |

## 4. Run 生命周期

1. Run API 验证 SSO 身份、Workspace、AgentVersion、输入引用和幂等键，创建 QUEUED Run。
2. Coordinator 获取租约并写入 worker epoch；Worker 校验版本、策略、预算和数据快照。
3. 外层图调用受控 Deep Agent 规划；计划经过工具、模型、数据和资源策略检查。
4. 检索、指标和事件分析可以并行；子 Agent 继承父 Run 的 tenant、scope、模型路由和预算。
5. 需要审批、编辑或补充信息时调用 interrupt，持久保存后释放 Worker。
6. 恢复时使用同一 thread_id 和声明的 payload；审批前代码必须幂等，外部副作用放在批准后的工具任务。
7. 输出经过证据、数值、密级、DLP 和引用检查，写入草稿/导出对象及最终状态。

## 5. 状态与一致性不变量

- PostgreSQL run 表使用 state_version compare-and-set；终态不可重开。
- QUEUED → RUNNING → WAITING_REVIEW/UNKNOWN/终态，状态迁移只能由服务端允许的边执行。
- 同一 approval_id 只能被一个执行意图消费；Outbox 重放沿用同一业务幂等键。
- 租约续期和状态提交携带 worker_epoch；旧 Worker 写入必须失败。
- Checkpoint 的 graph revision、AgentVersion、策略版本和 Run context 摘要必须匹配。
- 下游已执行而本地超时进入 UNKNOWN；恢复先查 operation_id、回执或对账，禁止盲重试。
- 客户端重试 POST /runs 使用 Idempotency-Key；事件以 event_seq/event_id 去重，after_seq 支持重连。

## 6. Deep Agents 沙箱与资源

首期允许使用 Deep Agent，但使用受限 profile：文件只在本 Run 工作区，默认无出网，工具仅来自 allowlist，子 Agent 不可提升权限。初始资源上限进入 AgentVersion，例如 2 vCPU、4 GiB 内存、2 GiB 工作区、15 分钟墙钟、4 层子 Agent、32 个子任务；最终值由纵向切片压测确认。镜像固定 digest，制品通过病毒、敏感信息、SBOM、签名和漏洞门禁。

MCP 只接入登记和固定版本服务器；HTTP MCP 通过代理校验 audience、tenant、scope、过期时间，stdio MCP 进程隔离并限制环境变量。Daytona 仅作为开发/受限沙箱，不持有生产凭据。

## 7. 失败与验收

验证 Worker 强杀、数据库提交后崩溃、Outbox 投递前崩溃、下游已执行而本地超时、审批期间版本发布、权限撤销、重复消息、租约过期和 LangSmith Cloud 不可用。验收要求不重复创建 Run、不重复消费审批、不盲目重放副作用；版本/策略/快照不兼容时终止并要求重新审批。

## 8. 待冻结决策

同步/异步边界、每类 Run 的模型/Token/工具/子 Agent/墙钟预算、队列 single-writer 方案、生产文件工具是否启用、LangSmith Trace 字段白名单与 outage 行为。
