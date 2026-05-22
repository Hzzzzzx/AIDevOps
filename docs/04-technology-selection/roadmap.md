# AIDevOps 改造路线图

> 状态：**草案** — 待用户确认最终选型后执行
>
> 更新：2026-05-21
>
> 前置条件：本文档包含 Mastra 路线和 Dify 路线两个方案，用户确认选型后择一执行。

---

## 方案 A：基于 Mastra 的改造路线

> 推荐。理由见 `adr-base-selection.md` §6。

### 总览

| 阶段 | 内容 | 周期 | 交付物 |
|------|------|------|--------|
| Phase 0 | 内核验证 | 2 周 | Mastra Agent + Tool + MCP 全验证 |
| Phase 1 | Agent 平台骨架 | 4 周 | Web 对话 + 多用户认证 + 基础管理后台 |
| Phase 2 | Agent 协作层 | 4 周 | SubAgent + ACP 协议 + DevOps 工具 |
| Phase 3 | 企业级能力 | 3 周 | 多租户隔离 + 审计追溯 + 权限模型 |
| Phase 4 | 生产化 | 3 周 | 部署运维 + 监控告警 + 性能优化 |

**总周期：~16 周（4 个月），1 人 + AI Agent 辅助**

### Phase 0：内核验证（第 1-2 周）

> 目标：验证 Mastra 全部核心能力，消除技术风险

| 任务 | 工时 | AI 承担 | 验收标准 |
|------|------|---------|---------|
| Workflow 引擎试用 | 2d | 50% | .then/.branch/.parallel + suspend/resume 跑通 |
| Memory 系统试用 | 1d | 60% | 对话记忆 + 语义检索 + 工作记忆 |
| MCP Server/Client 试用 | 2d | 40% | 自定义 MCP 工具注册 + 调用 |
| SubAgent 接口验证 | 2d | 50% | 模拟 ACP 协议调用外部 Agent |
| Storage 层测试 | 1d | 70% | PostgreSQL 存储对接 |
| 性能基准测试 | 1d | 40% | 单 Agent 对话延迟 < 2s (含 LLM) |

**里程碑**：Mastra 全核心能力验证完毕，确认无阻塞性 Gap。

### Phase 1：Agent 平台骨架（第 3-6 周）

> 目标：搭建可用的 Web 对话平台

| 任务 | 工时 | AI 承担 | 关键技术 |
|------|------|---------|---------|
| **Hono HTTP 服务** | 3d | 80% | 扩展 Demo 的 Hono 服务，加认证中间件 |
| **WebSocket/SSE 推送** | 2d | 70% | 流式对话响应 |
| **React 前端 - 对话工作台** | 5d | 85% | 复用 AgentCenter ConversationWorkbench 设计 |
| **React 前端 - Agent 配置页** | 3d | 80% | Agent 参数 + 工具绑定 + Prompt 编辑 |
| **React 前端 - 工具管理页** | 2d | 80% | MCP 工具注册/浏览/测试 |
| **用户认证** | 3d | 70% | JWT + OAuth2 (GitHub/企微) |
| **数据模型 + API** | 3d | 75% | User/Agent/Conversation/Tool CRUD |
| **基础管理后台** | 2d | 70% | 用户列表 + Agent 列表 + 使用统计 |

**参考文件**：
- Mastra Demo: `/Users/hzz/workspace/AIDevOps/mastra-demo/`
- AgentCenter 前端: `/Users/hzz/workspace/AgentCenter/agentcenter-web/`
  - 可复用 40%：对话 UI 组件、布局框架、状态管理模式

**里程碑**：内部可用的 Web 对话平台，用户登录 → 创建 Agent → 对话 → 管理工具。

### Phase 2：Agent 协作层（第 7-10 周）

> 目标：实现 AIDevOps 核心差异化——Agent 指挥 Agent

| 任务 | 工时 | AI 承担 | 关键技术 |
|------|------|---------|---------|
| **SubAgent 接口实现** | 3d | 60% | 扩展 Mastra SubAgent，定义 Agent-to-Agent 协议 |
| **ACP 协议适配器** | 5d | 50% | 对接 OpenCode / Codex / Claude Code |
| **Agent 编排引擎** | 5d | 40% | Agent 自主决策调用哪个 SubAgent |
| **DevOps 工具集** | 5d | 50% | CI/CD 触发 / 日志查询 / 部署 / 监控 |
| **工具执行沙箱** | 3d | 40% | 命令执行隔离 + 超时控制 |
| **协作对话 UI** | 3d | 75% | 展示多 Agent 协作过程 + 中间结果 |

**关键设计**：
```
用户对话 → 主 Agent (Mastra)
              ├── SubAgent: DevOps 工具 (内置)
              ├── SubAgent: 知识检索 (内置)
              ├── SubAgent: OpenCode (ACP 协议)
              ├── SubAgent: Codex (ACP 协议)
              └── SubAgent: [未来扩展...]
```

**里程碑**：用户可通过对话让主 Agent 自主编排调用外部编码 Agent。

### Phase 3：企业级能力（第 11-13 周）

> 目标：满足企业内部多用户使用

| 任务 | 工时 | AI 承担 | 关键技术 |
|------|------|---------|---------|
| **多租户隔离** | 3d | 50% | 项目级数据隔离 + 资源配额 |
| **RBAC 权限** | 3d | 50% | 参考 Mastra EE FGA 设计 |
| **审计追溯** | 2d | 60% | 全操作日志 + 对话历史追溯 |
| **SSO 集成** | 2d | 50% | 企业微信 / LDAP / SAML |
| **通知系统** | 1d | 70% | Agent 任务完成通知 (飞书/企微) |

**里程碑**：多用户隔离运行，权限管控，操作可追溯。

### Phase 4：生产化（第 14-16 周）

> 目标：可稳定运行

| 任务 | 工时 | AI 承担 | 关键技术 |
|------|------|---------|---------|
| **Docker 化部署** | 2d | 80% | Docker Compose / Helm Chart |
| **监控告警** | 2d | 60% | Prometheus + Grafana |
| **日志系统** | 1d | 70% | 结构化日志 + ELK |
| **性能优化** | 3d | 40% | 连接池 + 缓存 + 流式优化 |
| **安全加固** | 2d | 50% | CORS/CSP/Rate Limit/输入校验 |
| **文档 + 测试** | 2d | 70% | API 文档 + 集成测试 |

**里程碑**：生产级部署，监控覆盖，安全加固。

---

## 方案 B：基于 Dify 的改造路线

> 备选。Agent 核心弱但平台完整度高。

### 总览

| 阶段 | 内容 | 周期 | 交付物 |
|------|------|------|--------|
| Phase 0 | 裁剪瘦身 | 3 周 | 裁掉 ~310K LOC，~7 容器 |
| Phase 1 | 业务定制 | 3 周 | AgentCenter 页面 + DevOps 工具 |
| Phase 2 | Agent 增强 | 4 周 | SubAgent 扩展 + Agent 协作 |
| Phase 3 | 生产化 | 2 周 | 部署优化 + 安全加固 |

**总周期：~12 周（3 个月），1 人 + AI Agent 辅助**

### Phase 0：裁剪瘦身（第 1-3 周）

> 目标：460K LOC → ~150K LOC，12 容器 → 7 容器

| 任务 | 工时 | AI 承担 | 风险 |
|------|------|---------|------|
| **裁向量库** | 2d | 60% | Weaviate → pgvector，删 providers/vdb/ 15 家 |
| **裁 RAG 引擎** | 5d | 40% | 解耦 RAG 与 Agent 强耦合，高风险 |
| **裁无关 AppMode** | 3d | 70% | 删 COMPLETION / CHANNEL，保留 CHAT/AGENT_CHAT/WORKFLOW |
| **轻量部署** | 2d | 80% | 合并容器，Weaviate→pgvector，砍 SSRF proxy |
| **裁可观测集成** | 1d | 80% | 保留 1-2 种 trace，删其余 |
| **验证回归** | 2d | 50% | 确保裁剪后基本功能正常 |

**裁剪清单**：
```
可安全裁掉：
  ├── providers/vdb/ (15 家向量库，留 pgvector)  → 省 ~25K LOC
  ├── providers/trace/ (保留 1-2 种)              → 省 ~15K LOC
  ├── core/rag/ (大部分)                           → 省 ~10K LOC
  ├── AppMode: COMPLETION, CHANNEL                 → 省 ~5K LOC
  └── 无关 Celery 队列: mail, dataset_summary      → 省 ~2K LOC

需谨慎裁剪：
  ├── plugin_daemon (裁后需改回内置模型调用)        → 高风险
  └── sandbox (如不需要代码执行)                    → 中风险
```

**里程碑**：7 容器轻量部署，~150K LOC，基本对话功能正常。

### Phase 1：业务定制（第 4-6 周）

> 目标：加 AgentCenter 业务页面 + DevOps 工具

| 任务 | 工时 | AI 承担 | 说明 |
|------|------|---------|------|
| **看板页面** | 2-3d | 80% | ReactFlow 工作流可视化 |
| **首页概览** | 1-2d | 85% | 使用统计 + 快捷入口 |
| **项目上下文设置** | 2-3d | 75% | 项目级 Agent 配置 + 知识库绑定 |
| **后端 API** | 3-5d | 70% | 新增 Flask Blueprint + Service |
| **DevOps MCP 工具** | 3d | 50% | CI/CD/日志/部署 工具注册 |

**里程碑**：Dify 平台上已有 AIDevOps 业务定制，可内部试用。

### Phase 2：Agent 增强（第 7-10 周）

> 目标：补齐 Dify 最大短板——Agent 协作能力

| 任务 | 工时 | AI 承担 | 风险 |
|------|------|---------|------|
| **SubAgent 接口设计** | 3d | 40% | 在 `core/agent/` 上扩展，需深度理解 Dify 架构 |
| **ACP 协议适配** | 5d | 40% | Python 实现 ACP 客户端 |
| **Agent 编排逻辑** | 5d | 30% | 修改 Agent 策略，加自主决策能力 |
| **协作对话 UI** | 3d | 75% | 前端展示多 Agent 协作过程 |
| **集成测试** | 2d | 60% | 端到端验证 |

**关键挑战**：
- Dify Agent 策略是 FC/CoT 二选一，加自主编排需重构策略层
- `core/agent/` 只有 2.3K LOC，扩展空间大但也意味着基础薄弱
- 需要深度理解 `core/app/` 的 18.5K LOC 才能安全修改

**里程碑**：Agent 可通过 ACP 指挥外部编码 Agent。

### Phase 3：生产化（第 11-12 周）

| 任务 | 工时 | AI 承担 |
|------|------|---------|
| **安全加固** | 2d | 50% |
| **审计追溯** | 1d | 70% |
| **监控告警** | 1d | 80% |
| **部署优化** | 2d | 70% |

**里程碑**：生产级部署。

---

## 方案对比

| 维度 | 方案 A (Mastra) | 方案 B (Dify) |
|------|-----------------|---------------|
| **总周期** | 16 周 | 12 周 |
| **Agent 核心质量** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ → ⭐⭐⭐⭐ (改造后) |
| **平台完整度** | ⭐⭐ → ⭐⭐⭐⭐ (自建后) | ⭐⭐⭐⭐⭐ |
| **技术风险** | 低（加法） | 中（减法+改法） |
| **AI 辅助效率** | 高 (TS, 70-90%) | 中 (Python, 50-85%) |
| **长期可维护性** | 高 | 中 |
| **理念匹配** | 完美 (Agent-first) | 有冲突 (DAG vs 自主) |
| **生态依赖** | 低 (24K⭐) | 高 (141K⭐, 持续上游更新) |

## 决策建议

- **如果 AIDevOps 核心价值是 Agent 自主编排能力** → 方案 A (Mastra)
- **如果优先快速出平台原型，Agent 能力后期再补** → 方案 B (Dify)
- **如果团队 Python 经验丰富** → 方案 B 略有优势
- **如果团队偏好 TypeScript 全栈** → 方案 A 有明显优势
