# ADR: AIDevOps 平台底座选型

> 状态：**进行中** — Mastra + Dify 均已完成深度评估，待用户试用后做最终决策
> 
> 更新：2026-05-21

## 1. 决策背景

AIDevOps 是一个 AI 赋能 DevOps 的企业内部多用户平台。核心需求：

1. **对话式编排 Agent** — 用户通过 Web 对话与 Agent 交互
2. **可扩展核心** — Hook 机制 + Skill/MCP 工具生态
3. **外部 Agent 调度** — 通过 ACP 等协议指挥 OpenCode/Codex 等编码 Agent
4. **多用户项目隔离** — 后期加审计追溯
5. **Web 前端** — 有最好，没有自建

**选型策略**：选择一个开源底座 fork 后深度定制，而非从零构建。

## 2. 候选方案

经预筛 + 深度分析，入围两个最终候选：

| 维度 | Mastra | Dify |
|------|--------|------|
| **语言** | TypeScript (Node.js) | Python (Flask) + TS (Next.js) |
| **代码规模** | ~2,900 TS 文件 / 74MB | 854K LOC / 9,164 文件 |
| **License** | Apache 2.0 + EE (ee/ 目录) | Dify OSS (Apache 2.0+) |
| **Web UI** | ❌ 无（框架/SDK） | ✅ 完整平台（Next.js 15） |
| **MCP** | ✅ 一等公民 | ✅ 支持 |
| **Stars** | 24K | 141K |
| **改造量估算** | 2-3 月 | 4-6 月 |
| **深度分析排名** | 🥇 第 1 | 🥈 第 2 |

## 3. Mastra 试用评估

### 3.1 架构质量

```
Mastra 核心架构：
┌─────────────────────────────────────────┐
│  Mastra (DI 容器)                        │
│  ├── Agent (7,086 行核心)                │
│  │   ├── SubAgent 接口 ← 外部 Agent 调度 │
│  │   ├── MessageList                    │
│  │   └── StreamProcessor                │
│  ├── Workflow 引擎 (.then/.branch/.parallel) │
│  ├── Tools (createTool + Zod Schema)     │
│  ├── Memory (对话/语义/工作记忆)          │
│  ├── Storage (PG/LibSQL/Mongo/...)       │
│  └── MCP Server                         │
├── LLM 抽象层 (40+ Provider, 含智谱 ZAI)  │
└── HTTP Server (Hono)                     │
```

**评分**：

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | ⭐⭐⭐⭐⭐ | 严格 TS，Vitest 测试覆盖好，模块边界清晰 |
| 架构合理性 | ⭐⭐⭐⭐⭐ | DI 容器模式，Agent/Workflow/Tool/Memory 正交解耦 |
| 可扩展性 | ⭐⭐⭐⭐⭐ | Hook 系统 + SubAgent 接口 + MCP + Tool 注册 |
| SubAgent/外部调度 | ⭐⭐⭐⭐⭐ | 原生 SubAgent 接口，天然支持指挥外部 Agent |
| 文档 | ⭐⭐⭐⭐ | 官方文档完善，源码注释少但类型定义清晰 |
| 社区/生态 | ⭐⭐⭐ | 24K stars，YC W25，增长快但生态早期 |

### 3.2 实际验证

| 验证项 | 结果 |
|--------|------|
| 源码 clone + build | ✅ 核心包 OK (102/112) |
| Agent + createTool 基础测试 | ✅ 通过 |
| 智谱 GLM 支持 | ✅ 原生 provider `zai`（glm-4.5 ~ glm-5.1） |
| Demo 后端 (Hono HTTP + SSE) | ✅ 通过 |
| Demo 前端 (对话 UI) | ✅ 通过 |
| Workflow 引擎 | ⚠️ 未试用（下阶段） |
| Memory 系统 | ⚠️ 未试用（下阶段） |
| MCP Server | ⚠️ 未试用（下阶段） |

### 3.3 真实案例验证

| 客户 | 场景 | 效果 |
|------|------|------|
| **Replit** | Agent 3 — 造 Agent 的 Agent | $150M ARR，90% 自主率，200 分钟工作流 |
| **SoftBank** | 商务文档 AI 生成 | 文档创建：数小时 → 1-2 分钟 |
| **Sanity** | CMS 内容 Agent | 单次会话 227 处编辑 |
| **Factorial** | HR 平台 Agent | 遵循权限边界，零幻觉 |
| **WorkOS** | GTM 自动化 | 工程师数周上线 |

### 3.4 与 AIDevOps 需求映射

| AIDevOps 需求 | Mastra 对应能力 | Gap |
|---------------|-----------------|-----|
| 对话式 Agent | `Agent.generate()` / `Agent.stream()` | ✅ 完美匹配 |
| Hook/Skill 扩展 | `beforeToolCall` / `afterToolCall` 等 Hook | ✅ 完美匹配 |
| MCP 工具 | 内置 MCP Server/Client | ✅ 完美匹配 |
| 指挥外部 Agent (ACP) | `SubAgent` 接口 | ✅ 完美匹配，需实现 ACP SubAgent |
| 多用户隔离 | EE 版有 FGA/RBAC | ⚠️ 开源版无，需自建 |
| Web 前端 | 无 | ❌ 需自建（Demo 已验证可行性） |
| DevOps 集成 | — | ❌ 需自建工具 |
| 审计追溯 | — | ❌ 需自建 |

### 3.5 优势与风险

**优势**：
1. **SubAgent 接口是杀手锏** — 原生支持外部 Agent 调度，与 Replit 模式一致
2. **TypeScript 全栈** — 前后端统一，团队学习成本低
3. **架构精简** — 核心清晰，改造成本低（2-3 月）
4. **LLM Provider 丰富** — 40+ Provider，原生支持智谱
5. **团队背景** — Gatsby 原班人马，YC W25，$35M Series A
6. **耐用执行** — Workflow suspend/resume 支持人工审批场景

**风险**：
1. **无 Web UI** — 必须自建前端（但 Demo 验证了 Hono + HTML 可行）
2. **无多租户** — 开源版无 RBAC，需自建或参考 EE 实现
3. **生态早期** — 24K stars vs Dify 141K，社区插件少
4. **快速迭代** — API 可能频繁变动（每周发版）
5. **Python 生态弱** — 如果团队有 Python 工具链，集成需要桥接

## 4. Dify 试用评估

### 4.1 架构质量

```
Dify 完整架构（v1.x，2026-05-20 源码分析）：
┌──────────────────────────────────────────────────────────────────┐
│  Nginx (反向代理 + 路由)                                          │
│  ├── /console/api/* → Flask API (console Blueprint)               │
│  ├── /api/*         → Flask API (web Blueprint)                   │
│  ├── /v1/*          → Flask API (service_api Blueprint)           │
│  ├── /mcp/*         → Flask API (mcp Blueprint)                   │
│  ├── /e/*           → Plugin Daemon (独立 Go 服务)                 │
│  ├── /socket.io/*   → WebSocket (gevent)                          │
│  └── /*             → Next.js 15 (Web UI)                         │
├──────────────────────────────────────────────────────────────────┤
│  Flask API (gevent 同步, 7 Blueprint)                             │
│  ├── controllers/  (39.5K LOC) — HTTP 入口                        │
│  ├── services/     (45.4K LOC) — 业务逻辑                         │
│  ├── core/         (80.2K LOC) — 核心引擎                         │
│  │   ├── app/      (18.5K) — 7 种 AppMode 运行时                  │
│  │   │   ├── CHAT / AGENT_CHAT / ADVANCED_CHAT                    │
│  │   │   ├── WORKFLOW / COMPLETION                                │
│  │   │   ├── CHANNEL (聊天助手频道)                                 │
│  │   │   └── RAG_PIPELINE                                         │
│  │   ├── workflow/ (5.7K) — graphon DAG 引擎                      │
│  │   │   ├── 21 种节点类型                                         │
│  │   │   ├── 拓扑排序 + 线程池并行调度                               │
│  │   │   ├── VariablePool 数据总线                                  │
│  │   │   └── Layer 洋葱中间件 (限流/配额/可观测/持久化)              │
│  │   ├── tools/    (8.3K) — 工具注册/执行                          │
│  │   ├── mcp/      (4.7K) — MCP Server/Client                     │
│  │   ├── agent/    (2.3K) — Agent 策略 (FC + CoT)                 │
│  │   ├── plugin/   (5.9K) — Plugin Daemon 客户端                   │
│  │   └── rag/      (13.3K) — RAG 检索引擎                         │
│  ├── providers/    (46.3K) — 集成层                                │
│  │   ├── vdb/      (28.1K) — 16 种向量库                           │
│  │   └── trace/    (18.2K) — 10+ 种可观测                          │
│  ├── models/       (11.0K) — SQLAlchemy ORM                       │
│  └── tasks/        (8.6K) — Celery 异步任务                        │
├──────────────────────────────────────────────────────────────────┤
│  Celery Worker (4 worker, 9 队列)                                  │
│  ├── dataset / pipeline / priority_pipeline                       │
│  ├── mail / dataset_summary / workflow_draft_var                  │
│  └── app_deletion / schedule_executor                             │
├──────────────────────────────────────────────────────────────────┤
│  Plugin Daemon (独立 Go 服务)                                       │
│  ├── 插件隔离运行 (独立 venv)                                       │
│  └── 模型提供商 v1.14 全改插件机制                                   │
├──────────────────────────────────────────────────────────────────┤
│  Sandbox (Go/Gin, 进程级隔离)                                       │
│  ├── 代码执行沙箱 (Python/JS)                                       │
│  └── SSRF Proxy (Squid, 阻止内网访问)                               │
├──────────────────────────────────────────────────────────────────┤
│  数据层                                                            │
│  ├── PostgreSQL (主存储)                                            │
│  ├── Redis (缓存 + Celery Broker + 会话)                           │
│  ├── Weaviate (向量库, 可换 pgvector)                               │
│  └── MinIO (对象存储, 可选)                                         │
└──────────────────────────────────────────────────────────────────┘
```

**代码规模**：
- 后端 Python：~260K LOC（854 文件）
- 前端 TypeScript：~197K LOC
- 总计：~460K LOC / 9,164 文件
- 裁剪后估算：~150K LOC (33%)

**评分**：

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | ⭐⭐⭐ | 模块化尚可但历史包袱重，部分代码耦合（RAG 内嵌 Agent） |
| 架构合理性 | ⭐⭐⭐⭐ | 分层清晰（controller→service→core），但同步 Flask 限制并发 |
| 可扩展性 | ⭐⭐⭐⭐ | 插件机制 (v1.14) + MCP + 工具注册 + 21 种工作流节点 |
| SubAgent/外部调度 | ⭐⭐ | 无原生 SubAgent，Agent 核心仅 2.3K LOC，需自行扩展 |
| 文档 | ⭐⭐⭐⭐⭐ | 官方文档极其完善，API 文档 + 部署文档 + 插件开发指南 |
| 社区/生态 | ⭐⭐⭐⭐⭐ | 141K stars，300M+ 自部署，Volvo/Ricoh/AWS 等 40+ 500 强客户 |

### 4.2 实际验证

| 验证项 | 结果 |
|--------|------|
| Docker Compose 部署 | ✅ 12 容器全部 healthy，修复 3 个网络问题 |
| Web UI 访问 | ✅ Next.js 前端正常，`http://localhost` |
| 管理员注册/登录 | ✅ `331380069@qq.com` / Base64 密码 |
| 多租户架构验证 | ✅ Account→Tenant→TenantAccountJoin，5 级 RBAC |
| 工作流引擎分析 | ✅ graphon v0.4.0 (Dify 官方, Apache 2.0) |
| 沙箱隔离分析 | ✅ 三层防御，进程级隔离，冷启动 50-200ms |
| 源码全量分析 | ✅ 7 轮逐层扫描，controllers→services→core→providers |
| 智谱插件安装 | ⚠️ Marketplace 容器内访问被拦，需浏览器手动安装 |
| Agent 对话测试 | ⏳ 待智谱插件安装后测试 |
| MCP 功能测试 | ⏳ 待智谱插件安装后测试 |

### 4.3 企业客户验证

| 客户 | 场景 | 规模 |
|------|------|------|
| **Volvo** | 企业知识管理 Agent | 全球 500 强 |
| **Ricoh** | 内部文档 AI 助手 | 全球 500 强 |
| **AWS** | 云服务集成平台 | 全球 500 强 |
| **MongoDB** | 数据库操作 AI Agent | 上市公司 |
| **漫步者** | 智能客服 | 国内上市 |
| **360 数科** | 金融风控 Agent | 国内金融科技 |

数据来源：Dify 官网 + GitHub + 公开报道，共 40+ 500 强企业使用。

### 4.4 与 AIDevOps 需求映射

| AIDevOps 需求 | Dify 对应能力 | Gap |
|---------------|--------------|-----|
| 对话式 Agent | CHAT + AGENT_CHAT 模式 | ✅ 已有，但 Agent 核心薄（仅 FC/CoT） |
| Hook/Skill 扩展 | 工具注册 + 插件机制 | ✅ 插件生态成熟，MCP 支持 |
| MCP 工具 | 内置 MCP Server/Client | ✅ 已有 |
| 指挥外部 Agent (ACP) | 无 SubAgent 接口 | ❌ **最大 Gap**，需自行设计 Agent 协作层 |
| 多用户项目隔离 | 多租户 + 5 级 RBAC | ✅ 已有，tenant_id 隔离 |
| Web 前端 | Next.js 15 完整平台 | ✅ **最大优势**，开箱即用 |
| DevOps 集成 | — | ❌ 需自建工具 |
| 审计追溯 | trace 模块 (10+ 集成) | ⚠️ 有基础可观测，审计需扩展 |
| 自主编排 Agent | 工作流是声明式 DAG | ⚠️ 与"Agent 自主编排"理念冲突，需重构 |

### 4.5 优势与风险

**优势**：
1. **开箱即用的完整平台** — Web UI + 多租户 + 管理后台 + 监控，省 2-3 月前端/平台开发
2. **141K stars + 40+ 500 强** — 生态成熟，社区活跃，持续迭代有保障
3. **graphon DAG 引擎** — 自研 Apache 2.0，可自由修改扩展
4. **AgentCenter 60% 功能直接复用** — 只需加 2-3 个业务页面（看板/概览/上下文设置）
5. **多租户/RBAC** — 企业级隔离已内置，5 级权限控制
6. **插件生态** — v1.14 插件机制成熟，模型/工具/扩展均插件化

**风险**：
1. **Agent 核心薄弱** — `core/agent/` 仅 2.3K LOC，无 SubAgent/多 Agent 协作，需大幅扩展
2. **声明式 DAG vs 自主 Agent** — 工作流是预定义编排，与 AIDevOps "Agent 自主编排" 理念冲突
3. **Python + Flask 同步架构** — gevent 协程限制并发，不如 TS 异步模型
4. **RAG 强耦合** — Agent 里 RAG retrieval 内嵌，裁剪需解耦
5. **460K LOC 代码量** — 学习成本高，裁剪工作量大（~310K LOC 需裁掉）
6. **v1.14 插件机制** — 裁 plugin_daemon 后需把模型调用改回内置

## 5. 对比矩阵

| 维度 | Mastra | Dify | 权重 | 说明 |
|------|--------|------|------|------|
| **Agent 对话核心** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 高 | Mastra Agent 7K LOC vs Dify 2.3K LOC |
| **外部 Agent 调度** | ⭐⭐⭐⭐⭐ | ⭐⭐ | 高 | Mastra 原生 SubAgent vs Dify 无 |
| **MCP/工具扩展** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 高 | 两者均支持，Mastra 更原生 |
| **Web UI** | ❌ 需自建 | ✅ 完整平台 | 中 | Dify 最大优势，省 2-3 月 |
| **多租户/RBAC** | ⚠️ 需自建 | ✅ 5 级 RBAC | 中 | Dify 内置企业级隔离 |
| **代码可维护性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 中 | Mastra 精简 2.9K 文件 vs Dify 9K 文件 |
| **改造量** | 2-3 月 | 3-4 月（渐进式） | 中 | Dify 裁剪+扩展 vs Mastra 建平台 |
| **工作流理念** | Agent 自主 | 声明式 DAG | 高 | Mastra 符合"Agent 自主编排"理念 |
| **技术栈** | TypeScript | Python + React | 低 | 用户已接受 Dify 技术栈 |
| **社区生态** | 24K⭐ YC W25 | 141K⭐ 40+ 500强 | 低 | Dify 生态远更成熟 |

### 加权评分（5 分制）

| 维度 | 权重 | Mastra | Dify 加权 | Dify |
|------|------|--------|----------|------|
| Agent 对话核心 | 3x | 5 → 15 | 3 → 9 | 9 |
| 外部 Agent 调度 | 3x | 5 → 15 | 2 → 6 | 6 |
| MCP/工具扩展 | 3x | 5 → 15 | 4 → 12 | 12 |
| Web UI | 2x | 0 → 0 | 5 → 10 | 10 |
| 多租户/RBAC | 2x | 1 → 2 | 5 → 10 | 10 |
| 代码可维护性 | 2x | 5 → 10 | 3 → 6 | 6 |
| 工作流理念 | 3x | 5 → 15 | 2 → 6 | 6 |
| 社区生态 | 1x | 3 → 3 | 5 → 5 | 5 |
| **总计** | **19x** | **75** | | **64** |

## 6. 决策

### 分析

**Mastra 优势集中在"Agent 核心能力"**：SubAgent 接口、自主编排、Hook 扩展、TypeScript 全栈。这正是 AIDevOps 最核心的需求——一个能指挥其他 Agent 的 Agent 内核。

**Dify 优势集中在"平台完整度"**：Web UI、多租户、RBAC、管理后台、插件生态。这些都是"基础设施"，Mastra 需要从零搭建。

**关键矛盾**：
1. **Agent 自主编排 vs 声明式 DAG** — 用户明确表示"将来的工作流应该让 Agent 来把控"，Dify 的 graphon 是预定义 DAG，与这个理念有结构性冲突
2. **SubAgent 缺失** — Dify `core/agent/` 仅 2.3K LOC，无多 Agent 协作机制，这是 AIDevOps "指挥外部编码 Agent" 的核心能力
3. **改造方向不同** — Mastra 是"在优秀内核上建平台"（加法），Dify 是"在臃肿平台上改核心"（减法+改法）

### 推荐：**Mastra**

**理由**：
1. **Agent 核心是灵魂** — AIDevOps 的差异化价值在于 Agent 自主编排能力，Mastra 在这层远优于 Dify
2. **改造方向更安全** — 在精简内核上做加法（建平台）比在臃肿系统上做减法（裁核心）风险更低
3. **SubAgent 是杀手锏** — Replit 已验证这条路可行（$150M ARR, 90% 自主率）
4. **TypeScript 全栈** — 前后端统一，AI Agent 辅助重构效率高（70-90%）
5. **理念对齐** — Mastra 的 Agent-first 设计与 AIDevOps "Agent 自主编排" 完全一致

**接受的风险**：
- 需自建 Web UI（但 Demo 已验证 Hono + React 可行，AgentCenter 可复用 40%）
- 需自建多租户（但可参考 Mastra EE 的 FGA/RBAC 设计）
- 生态早期（但 24K stars + YC W25 + $35M 融资，增长迅猛）

### 备选路线

如果团队更倾向"快速出平台原型"而非"Agent 核心极致优化"：
- **选 Dify**，接受 Agent 核心薄弱的现实，后期在 `core/agent/` 上扩展 SubAgent 能力
- 改造路线：裁剪 → 加 AgentCenter 页面 → 加 Agent 协作层 → 局部优化
- 3-4 月出可用版本

## 7. 后续步骤

1. [x] 用户启动 Dify Docker Compose
2. [x] Dify 全维度深度分析（7 轮源码分析）
3. [x] 补充 Dify 评估数据到本文档
4. [x] 完成对比矩阵与决策分析
5. [ ] **用户确认最终选型**
6. [ ] 用户浏览器安装智谱插件 + 试用 Dify 对话
7. [ ] 输出改造路线图（基于选定方案）
