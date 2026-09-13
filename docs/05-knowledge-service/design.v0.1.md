# 知识服务能力设计

| 属性 | 内容 |
|---|---|
| 文档 ID | CAP-05_KNOWLEDGE_SERVICE |
| 版本 | v0.1 |
| 日期 | 2026-09-13 |
| 状态 | 拆分初稿；继承总蓝图 P0 未通过状态 |
| 上一版本 | [总蓝图 v0.2](../00-architecture/financial-agent-platform-blueprint.v0.2.md) |
| 依据 | [能力目录 v0.1](../catalog.v0.1.md)、总蓝图 v0.2、用户提供的能力表 |
| 责任团队 | 知识服务责任团队 |
| 本次变更 | 首次从总蓝图拆出本模块；不覆盖任何历史文档 |

## 1. 范围与边界

本模块覆盖：内容接入、ACL 同步、时间过滤、OSS 对象、混合检索、重排、引用和删除 lineage。

依赖：总蓝图、Run 服务、平台管理、知识服务、工具网关、模型网关或观测与审计（按本模块职责取其必要部分）。

本模块不能改变 Tenant、Workspace、数据密级、AgentVersion、审批、审计或生产准入规则；共用约束以总蓝图 v0.2 第 0 节和第 13 节为准。

## 2. 接口与交接

- POST /knowledge/ingestions 返回对象版本；POST /knowledge/search 使用 server-side principal 和 effective_at；deletions 创建 tombstone 与 lineage 任务。

## 3. 验收条件

- 验证撤权传播上限、缓存 TTL、索引延迟、未来资料隔离、跨 Workspace 越权和签名 URL 过期。

## 4. 版本记录

| 版本 | 日期 | 变更 |
|---|---|---|
| v0.1 | 2026-09-13 | 首次能力拆分；待 P0 门禁和详细设计评审 |
