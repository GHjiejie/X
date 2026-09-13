# 金融 Agent 平台设计文档目录

| 属性 | 内容 |
|---|---|
| 文档 ID | DOC-CATALOG |
| 版本 | v0.2 |
| 日期 | 2026-09-13 |
| 上一版本 | [能力目录 v0.1](catalog.v0.1.md) |
| 本次变更 | 增加控制平面、执行平面、数据与业务服务区三份详细设计；保留 v0.1 不变 |

## 架构分区详细设计

| 分区 | 文档 |
|---|---|
| 控制平面 | [控制平面详细设计 v0.1](00-architecture/01-control-plane/control-plane.v0.1.md) |
| 执行平面 | [执行平面详细设计 v0.1](00-architecture/02-execution-plane/execution-plane.v0.1.md) |
| 数据与业务服务区 | [数据与业务服务区详细设计 v0.1](00-architecture/03-data-business-services/data-business-services.v0.1.md) |

三份文档分别拆解总体结构图中的治理与发布、任务执行与运行时、数据存储与金融业务服务边界，并给出组件职责、关键流程、接口与数据、失败恢复、安全控制、验收条件和待冻结决策。

## 能力设计文档

以下能力文档沿用 [能力目录 v0.1](catalog.v0.1.md) 中的版本，不在本次更新中覆盖：

- [员工工作台 v0.1](01-employee-workbench/design.v0.1.md)
- [平台管理 v0.1](02-platform-management/design.v0.1.md)
- [Run 服务 v0.1](03-run-service/design.v0.1.md)
- [Agent Runtime v0.1](04-agent-runtime/design.v0.1.md)
- [知识服务 v0.1](05-knowledge-service/design.v0.1.md)
- [工具网关 v0.1](06-tool-gateway/design.v0.1.md)
- [模型网关 v0.1](07-model-gateway/design.v0.1.md)
- [观测与审计 v0.1](08-observability-audit/design.v0.1.md)

## 总体蓝图

- [金融 Agent 平台总蓝图 v0.2 快照](00-architecture/financial-agent-platform-blueprint.v0.2.md)
- 原始总蓝图：[financial-agent-platform-blueprint.md](financial-agent-platform-blueprint.md)

版本规则见 [versioning v0.1](versioning.v0.1.md)。任何后续设计更新都应新增带版本号的文件或目录索引，不修改既有版本。
