# 控制平面详细设计

| 属性 | 内容 |
|---|---|
| 文档 ID | ARCH-CONTROL-PLANE |
| 版本 | v0.1 |
| 日期 | 2026-09-13 |
| 状态 | 分区详细设计初稿；继承总蓝图 P0 未通过状态 |
| 上一版本 | [总蓝图 v0.2](../financial-agent-platform-blueprint.v0.2.md) |
| 依据 | [能力目录 v0.1](../../catalog.v0.1.md)、总蓝图 v0.2、平台能力表 |
| 责任团队 | 平台治理与发布团队 |
| 本次变更 | 首次从总体结构图拆出控制平面；不覆盖既有文档 |

## 1. 目标与边界

控制平面回答“谁可以发布什么能力、什么数据和工具可被使用、哪个版本可以进入执行平面”。它管理配置、策略和审批，不执行 Agent 任务，不直接持有业务数据写权限，也不替代机构 IAM、法务和风险审批。

## 2. 结构设计

~~~mermaid
flowchart LR
    I[SSO 与组织] --> W[Tenant / Workspace]
    W --> R[Agent Prompt Tool Model 注册]
    R --> E[LangSmith 数据集与门禁]
    E --> P[发布审批与灰度]
    P --> M[签名不可变 Manifest]
    M --> X[执行平面]
    W --> Q[配额与策略]
    Q --> X
    P --> A[发布/撤销审计]
~~~

每个方框可以是同一平台服务中的模块。控制平面向执行平面只交付不可变版本清单、策略摘要、配额和撤销事件；不通过模型输出传递权限。

## 3. 组件职责

| 组件 | 详细职责 | 明确不负责 |
|---|---|---|
| Identity Adapter | 对接 SSO、MFA、组织、岗位、离职状态和 break-glass；解析可信声明 | 不复制长期凭据，不自行维护员工主数据 |
| Tenant/Workspace Registry | 定义机构租户、业务域、资料范围、密级、配额、负责人和分享边界 | 不因平台管理员身份授予业务正文权限 |
| Artifact Registry | 管理 Agent 图、Deep Agent profile、Prompt、middleware、Tool Schema、Model profile、知识 manifest 和计算函数 | 不让用户上传任意生产代码或动态安装依赖 |
| Policy Service | 计算资源、数据、模型、工具、审批、出网和配额策略，返回 policy_decision_id、原因码、有效期 | 不把规则写入 Prompt 供模型自行遵守 |
| Evaluation Gate | 管理 LangSmith Cloud 数据集、实验、人工标注和阻断规则；按当前 Developer/Plus 能力启用 | 不把 LangSmith Enterprise 专属能力当作必需条件 |
| Release Controller | 生成签名 manifest，执行灰度、停用、回滚、撤销和兼容性检查 | 不修改已被 Run 引用的版本 |
| Admin Audit View | 以最小权限展示配置和事件摘要，记录查询行为 | 不绕过 Workspace、Trace 或正文访问边界 |

## 4. 版本与发布流程

1. 开发者提交候选 AgentVersion，声明 Deep Agent profile、工具、模型、知识和计算依赖。
2. Registry 解析依赖并生成内容哈希；Policy Service 检查 Workspace、数据密级、供应商和权限。
3. Evaluation Gate 使用固定黄金集运行引用、数值、越权、注入、恢复、成本和输出格式评测。
4. 业务负责人、数据所有者、风险/安全和发布人完成职责分离审批；发布生成签名 manifest。
5. Release Controller 将版本灰度到指定 Workspace；任一阻断指标触发停止新 Run 和兼容版本回滚。
6. 工具、模型、资料或角色撤销时生成不可变撤销事件，通知执行、知识、工具和模型服务。

## 5. 核心契约

- AgentVersion 必须绑定 code digest、Deep Agent profile、Prompt digest、policy version、IAM/ACL snapshot、model profile、tool versions、corpus manifest、index version、metric definition 和评测实验。
- 发布别名只能解析到一个具体版本；历史 Run 保存具体版本，不随别名改变。
- 策略决定只能由服务端生成。客户端和模型不得改变 tenant、scope、密级、预算、工具权限或审批期限。
- 供应商 Endpoint、模型、地域、留存、脱敏和 LangSmith 项目配置进入版本清单；不能仅通过修改环境变量切换生产路由。
- 控制平面事件包含操作者、Workspace、版本、策略决定、审批和撤销关联，正文访问和密钥读取另行授权。

## 6. 失败与恢复

策略服务不可用时拒绝敏感数据和写操作；发布服务不可用不影响已经发布版本的运行。发现模型供应商、工具 Schema、ACL 或数据处理策略变化时，冻结受影响版本，不能自动降级门禁。恢复控制平面数据库时验证 manifest 哈希、审批链、版本依赖和撤销事件序号。

## 7. 验收条件

- 未评测、未批准、已停用或依赖不完整的版本无法进入执行平面。
- 修改 Prompt、模型、工具、知识 manifest、ACL 策略或计算函数会生成新版本。
- 管理员无法绕过 Workspace 边界；break-glass 需要双人批准、短时权限、原因和完整审计。
- 灰度、回滚和停用可在没有模型参与的情况下执行。
- 目录只引用具体版本，历史目录和历史设计文件内容不变。

## 8. 待冻结决策

司法辖区和内部政策、Tenant/Workspace 分享与 break-glass、Developer/Plus LangSmith Cloud 当前额度与保留能力、签名制品和 SBOM 门禁、P0 数据处理矩阵及各角色责任人。
