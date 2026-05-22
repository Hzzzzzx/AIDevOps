# AgentCenter 穿刺版回顾

> AIDevOps 项目的前身原型（AgentCenter）功能梳理与复盘，为新底座选型和迁移提供参考。

## 1. 项目概况

| 项 | 值 |
|---|---|
| 项目路径 | `/Users/hzz/workspace/AgentCenter/` |
| 定位 | AI Agent + DevOps 工作流平台的早期穿刺验证 |
| 技术栈 | Spring Boot 3.4 + MyBatis + SQLite / Vue 3 + Pinia + TypeScript |
| AI 引擎 | OpenCode Runtime（通过 SSE/HTTP 通信） |
| 通信方式 | REST + SSE（Server-Sent Events） |
| 数据库迁移 | Flyway（V1-V23，共 23 个版本） |
| 端口 | 4097（OpenCode）/ 8080（Bridge）/ 5173（Vue） |

## 2. 架构

```
浏览器(Vue3:5173)
    │ REST + SSE
    ▼
Java Bridge(:8080)  ←── Spring Boot + DDD分层
    │ SSE/HTTP
    ▼
OpenCode Serve(:4097)  ←── AI Agent 运行时
```

### 后端分层（DDD）

```
Controller → Application → Domain → Infrastructure
     ↓            ↓           ↓           ↓
   API入口    业务编排     领域对象    持久化/运行时
```

## 3. 数据模型（23 版迁移演进）

### 核心实体（V1 初始 15 表）

| 表 | 用途 |
|----|------|
| `user_account` | 用户账户 |
| `project_member` | 项目成员 |
| `work_item` | 工作项（TASK/USER_STORY/BUG/VULN 等） |
| `workflow_definition` | 工作流定义 |
| `workflow_node_definition` | 工作流节点定义 |
| `workflow_instance` | 工作流实例 |
| `workflow_node_instance` | 节点实例 |
| `agent_session` | AI 会话 |
| `agent_message` | AI 消息 |
| `runtime_event` | 运行时事件 |
| `artifact` | 产物（PRD/HLD/LLD 文档） |
| `confirmation_request` | 确认请求 |
| `confirmation_action` | 确认动作 |
| `skill_definition` | 技能定义 |
| `outbox_event` | 事件发件箱 |

### 后续演进（V2-V23 新增）

| 版本范围 | 新增内容 |
|----------|----------|
| V2-V10 | 工作流节点增强、会话扩展 |
| V11-V15 | 运行时资源管理、产物类型扩展 |
| V16-V20 | Skill 版本管理、MCP 集成 |
| V21-V23 | 项目上下文 Provider、工作流项目作用域 |

### 关键枚举

```
SessionType:     GENERAL, WORK_ITEM
MessageRole:     USER, ASSISTANT, SYSTEM, TOOL
WorkItemType:    FE, US, TASK, WORK, BUG, VULN
SkillStatus:     ENABLED, DISABLED, INVALID, UPDATING
WorkflowStatus:  RUNNING, READY, WAITING_CONFIRMATION, FAILED, COMPLETED, WORKFLOW_COMPLETED
ConfirmationActionType: ENTER_SESSION, APPROVE, REJECT, SUPPLEMENT, CHOOSE, RETRY, SKIP, ADVANCE
```

## 4. API 端点

| 端点 | 功能 |
|------|------|
| `/api/agent-sessions` | 会话管理（CRUD + 消息发送） |
| `/api/work-items` | 工作项管理（CRUD） |
| `/api/workflow-definitions` | 工作流定义 |
| `/api/workflow-instances` | 工作流实例（继续/重试/跳过） |
| `/api/confirmations` | 确认请求（解决/拒绝） |
| `/api/artifacts` | 产物管理 |
| `/api/runtime-resources` | 运行时资源 |
| `/api/runtime-events/stream` | SSE 事件流 |
| `/api/project-data-providers` | 项目数据提供者 |
| `/health` | 健康检查 |

## 5. 功能清单

### 5.1 已完成（需迁移到新底座）

#### 对话工作台 ★★★★★

- **关键文件**: `ConversationWorkbench.vue`（2777 行）、`MessageList.vue`、`ConversationInteractionBar.vue`
- **功能**: 多会话管理、实时消息流（SSE）、Markdown 渲染、执行步骤展示、工具调用内联显示、AI 回复分段渲染
- **后端**: `AgentSessionController` + `AgentSessionService`，完整的会话生命周期管理
- **迁移价值**: 核心交互模式已验证，UI 组件可参考复用

#### 工作流编排 ★★★★☆

- **关键文件**: `WorkflowCommandService.java`、`WorkflowPromptComposer.java`、`WorkflowNodeTransitionGuard.java`
- **功能**: 多节点工作流定义、串行执行、节点状态机（RUNNING → WAITING_CONFIRMATION → COMPLETED → WORKFLOW_COMPLETED）
- **前端**: `WorkflowConfig.vue`（配置）、`WorkflowTimeline.vue`（可视化）、`WorkflowNodeControlBar.vue`（对话中控制）
- **特色**: 支持 InputPolicy（输入路由策略）、Prompt 组合（每个节点可定制 Prompt）
- **迁移价值**: 核心业务逻辑，新底座若无工作流能力则需整体迁移

#### Skill 管理 ★★★★☆

- **关键文件**: `SkillManagement.vue`（420 行）、`SkillRegistryService.java`、`OpenCodeSkillFileService.java`
- **功能**: 上传 Skill ZIP、启用/禁用/验证、版本追踪、目录浏览
- **存储**: `runtime-workspace/.opencode/skills/` 下 Markdown 文件
- **已配置的 Skill**: PRD 设计、HLD 设计、LLD 设计、FE 用户故事生成、代码审查、安全扫描、API 设计
- **迁移价值**: Skill 加载/管理机制可直接复用设计思路

#### MCP 管理 ★★★★☆

- **关键文件**: `McpManagement.vue`（384 行）、`McpRegistryService.java`、`OpenCodeMcpFileService.java`
- **功能**: 导入 MCP Server 配置、启用/禁用、健康监控（OK/FAILED/UNKNOWN）、工具计数、连接测试
- **迁移价值**: MCP 集成模式已验证，可直接复用

#### 产物捕获 ★★★★☆

- **关键文件**: `ArtifactCaptureService.java`、`ArtifactViewer.vue`
- **功能**: 从 Agent 输出中解析 `AGENTCENTER_ARTIFACT` 块、Markdown 产物预览、代码高亮、文件引用
- **迁移价值**: 产物系统是差异化功能，需保留

#### 确认机制 ★★★★☆

- **关键文件**: `ConfirmationService.java`、`ConfirmationCard.vue`、`ConfirmationPanel.vue`
- **功能**: 人工确认门，支持 APPROVE/REJECT/SKIP/RETRY/CHOOSE/SUPPLEMENT/ADVANCE
- **迁移价值**: 安全关键功能，生产环境必须

#### 工作项看板 ★★★☆☆

- **关键文件**: `BoardView.vue`、`WorkItemCard.vue`、`BoardNodeCard.vue`
- **功能**: 看板视图、工作项 CRUD、类型分类（FE/US/TASK/BUG/VULN）
- **迁移价值**: 基础功能，新底座可能有更成熟的方案

### 5.2 半成品 / 只有骨架

#### 用户体系

- **现状**: 数据库有 `user_account`、`project_member` 表，但 **无登录/鉴权 API**
- **缺失**: 用户注册/登录、Session/Token 管理、权限控制、用户 Profile
- **新底座要求**: **必须有**，AIDevOps 的企业级多租户基础

#### 项目隔离

- **现状**: V22 迁移添加了 `workflow_definition_project_scope`，`ProjectDataProvider` 接口定义了数据隔离规范
- **缺失**: 完整的项目级数据隔离、跨项目访问控制
- **新底座要求**: **必须有**，用户间 + 项目间严格隔离

#### 知识 / RAG

- **现状**: `ProjectDataProvider` 接口 + `FixtureAlphaProjectDataProvider`（仅测试 Fixtures）
- **缺失**: 向量检索、RAG 管道、知识库管理
- **新底座要求**: **中期需要**，取决于业务需求

### 5.3 未建设

| 功能 | 现状 | 优先级 |
|------|------|--------|
| DevOps 集成（Git/CI-CD/监控） | 仅有调研文档，无代码 | P1（AIDevOps 核心） |
| 外部通知（Webhook/邮件/飞书） | 无代码 | P2 |
| 文件上传 | 无代码 | P2 |
| 完整多租户 | 无代码 | P1 |
| 审计追溯 | `runtime_resource_audit` 表存在但功能不完整 | P2 |

## 6. OpenCode Runtime 适配器

这是穿刺版最核心的基础设施，将 Java Bridge 与 OpenCode 运行时连接：

| 文件 | 职责 |
|------|------|
| `OpenCodeRuntimeAdapter.java` | 核心适配器，统一运行时接口 |
| `OpenCodeProcessManager.java` | OpenCode 进程生命周期管理 |
| `OpenCodeEventSubscriber.java` | SSE 事件订阅 |
| `OpenCodeCliClient.java` | CLI 客户端 |
| `OpenCodeSkillFileService.java` | Skill 文件操作 |
| `OpenCodeMcpFileService.java` | MCP 配置文件操作 |
| `RuntimeWorkspace.java` | 工作空间目录管理 |
| `OpenCodeRuntimeEventTranslator.java` | 事件翻译（OpenCode → Bridge 领域事件） |
| `OpenCodeTranslationState.java` | 翻译状态机 |
| `OpenCodeHttpCommandTransport.java` | HTTP 命令传输 |
| `OpenCodeSseEventStreamTransport.java` | SSE 事件流传输 |

**通信模式**:
- 命令下发: HTTP POST → OpenCode API
- 事件回传: SSE 长连接 → 事件翻译 → 前端推送

## 7. 对新底座选型的影响

### P0 必须满足（无则直接排除）

| 能力 | 原因 |
|------|------|
| 对话式 Agent 核心 | 穿刺版核心交互模式，必须有 |
| Skill + MCP 机制 | 已验证可行，生态基础 |
| Web 前端 | 企业级产品必须，最好自带 |
| 可 Fork 深度定制 | 底座策略要求 |

### P1 重要（影响选型权重）

| 能力 | 原因 |
|------|------|
| 工作流编排 | 穿刺版核心业务逻辑 |
| 外部 Agent 调度（ACP/API） | 指挥 OpenCode/Codex 的核心需求 |
| 多用户/项目隔离基础 | 企业级底线 |
| 产物管理 | 差异化功能 |

### P2 加分项

| 能力 | 原因 |
|------|------|
| 确认/审批机制 | 安全功能 |
| 知识库/RAG | 中期需求 |
| 外部通知通道 | 运维需求 |
| DevOps 工具集成 | 长期核心 |

## 8. 前端组件复用参考

穿刺版 Vue3 前端有大量可参考的组件实现：

| 组件 | 行数 | 复用价值 |
|------|------|----------|
| `ConversationWorkbench.vue` | 2777 | 对话交互核心，设计思路直接复用 |
| `SkillManagement.vue` | 420 | Skill 管理 UI 模式 |
| `McpManagement.vue` | 384 | MCP 管理 UI 模式 |
| `WorkflowConfig.vue` | - | 工作流配置交互 |
| `BoardView.vue` | - | 看板视图模式 |

**注意**: 复用时需考虑新底座的前端框架可能不同（React vs Vue），但交互模式和业务逻辑可跨框架复用。

## 9. 经验教训

1. **DDD 分层是正确的**: Controller → Application → Domain → Infrastructure 的分层在演进中表现良好，新底座应保留类似分层
2. **SSE 通信模式成熟**: 实时事件推送用 SSE 是正确选择，比 WebSocket 轻量且足够
3. **SQLite 不可生产**: 开发方便但生产必须换 PostgreSQL/MySQL
4. **用户体系应早建**: 穿刺版跳过了认证，导致后期加项目隔离很痛苦
5. **Skill 文件系统可行**: Markdown 文件管理 Skill 的方式简单有效，新底座可保留
6. **Flyway 迁移管理规范**: 23 个版本的有序迁移值得保留

---

*文档生成时间: 2026-05-20*
*基于 AgentCenter 穿刺版代码分析*
