# 工具网关能力设计

| 属性 | 内容 |
|---|---|
| 文档 ID | CAP-06_TOOL_GATEWAY |
| 版本 | v0.1 |
| 日期 | 2026-09-13 |
| 状态 | 拆分初稿；继承总蓝图 P0 未通过状态 |
| 上一版本 | [总蓝图 v0.2](../00-architecture/financial-agent-platform-blueprint.v0.2.md) |
| 依据 | [能力目录 v0.1](../catalog.v0.1.md)、总蓝图 v0.2、用户提供的能力表 |
| 责任团队 | 工具网关责任团队 |
| 本次变更 | 首次从总蓝图拆出本模块；不覆盖任何历史文档 |

## 1. 范围与边界

本模块覆盖：工具注册、参数校验、身份委托、限流、沙箱、执行确认、MCP 和业务动作控制。

依赖：总蓝图、Run 服务、平台管理、知识服务、工具网关、模型网关或观测与审计（按本模块职责取其必要部分）。

本模块不能改变 Tenant、Workspace、数据密级、AgentVersion、审批、审计或生产准入规则；共用约束以总蓝图 v0.2 第 0 节和第 13 节为准。

## 2. 接口与交接

- authorize 返回 policy_decision_id、canonical_args_hash 和审批要求；execution 要求幂等键、approval_id（如适用）和 worker_epoch；回执保存 operation_id。

## 3. 验收条件

- 验证错误 audience、跨租户资源、参数篡改、下游 UNKNOWN、工具撤销和 MCP 固定版本；禁止任意数据库、Shell 和网络。

## 4. 版本记录

| 版本 | 日期 | 变更 |
|---|---|---|
| v0.1 | 2026-09-13 | 首次能力拆分；待 P0 门禁和详细设计评审 |
