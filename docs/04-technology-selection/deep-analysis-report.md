# AIDevOps 底座选型 — 深度分析报告

> 对 5 个入围项目（Dify / Langflow / CrewAI / Mastra / Flowise）的架构拆解、可扩展性评估、Fork 可行性分析和改造量估算。

## 1. 基础数据

| 项目 | 语言 | 代码规模 | License | 有Web UI | 有MCP | Stars |
|------|------|----------|---------|---------|-------|-------|
| **Dify** | Python(Flask) + TS(Next.js) | 854K LOC / 9164 文件 | Dify OSS (Apache 2.0+) | ✅ | ✅ | 141K |
| **Langflow** | Python(FastAPI) + TS(React) | ~96MB 源码 | MIT | ✅ | ✅ | 148K |
| **CrewAI** | 纯 Python | 248K LOC / 1178 文件 | MIT | ❌ | ✅ | 51K |
| **Mastra** | TypeScript(Node.js) | 2893 TS 文件 / 74MB | Apache 2.0 + EE | ❌ | ✅ | 24K |
| **Flowise** | TS(Express) + React | ~280K LOC 估计 | Apache 2.0 | ✅ | ✅ | 52K |

## 2. 架构拆解

### 2.1 Dify

```
dify/
├── api/                    # Python/Flask 后端 (659K LOC)
│   ├── core/agent/         # Agent 运行时 (FC + CoT Runner)
│   ├── core/tools/         # 工具系统 (builtin/custom/mcp/plugin)
│   ├── core/workflow/      # 可视化工作流引擎
│   ├── core/mcp/           # MCP 客户端/服务器
│   ├── core/rag/           # RAG 管道
│   ├── services/           # 74 个业务服务
│   ├── models/             # 27 个 SQLAlchemy 模型
│   └── tasks/              # 55 个 Celery 异步任务
├── web/                    # Next.js 15 + React 19 前端 (195K LOC)
├── dify-agent/             # 独立 Agent 运行时
└── docker/                 # Docker Compose (10 个服务)
```

**关键特点**：
- Agent 执行器：`FunctionCallAgentRunner` → LLM 调用 → `ToolEngine` 执行工具 → 循环
- 工具注册：5 种 Provider (Builtin/Custom/MCP/Plugin/Workflow)
- 数据库：PostgreSQL + Redis + Celery
- 依赖重：PostgreSQL, Redis, Weaviate, MinIO, Nginx, SSRF Proxy...

### 2.2 Langflow

```
langflow/
├── src/
│   ├── lfx/                # 执行器核心
│   │   ├── base/agents/    # Agent 运行时 (LangChain AgentExecutor)
│   │   ├── components/     # 100+ 内置组件
│   │   ├── graph/          # 图执行引擎
│   │   ├── events/         # EventManager 事件系统
│   │   └── mcp/            # MCP 实现
│   ├── backend/base/       # FastAPI 平台层
│   │   ├── api/v1/         # ~37 个路由
│   │   ├── api/v2/         # v2 API
│   │   ├── services/       # 30+ 服务
│   │   └── alembic/        # DB 迁移
│   └── frontend/           # React 19 + Vite + @xyflow/react
```

**关键特点**：
- 三层架构：`lfx`(运行时) → `langflow-base`(平台) → `langflow`(发行)
- Agent 依赖 LangChain AgentExecutor
- 可视化构建器深度绑定（Flow JSON 是核心数据格式）
- EventManager 支持事件钩子

### 2.3 CrewAI

```
crewai/ (monorepo)
├── lib/
│   ├── crewai/             # 核心框架 (30MB)
│   │   ├── crew.py         # 2309 行 — Crew 编排
│   │   ├── task.py         # 1468 行 — Task 定义
│   │   ├── llm.py          # 2574 行 — LLM 集成
│   │   ├── agents/         # Agent 执行器
│   │   ├── flow/           # 事件驱动工作流 (@start/@listen/@router)
│   │   ├── memory/         # LanceDB 向量存储
│   │   ├── mcp/            # MCP 客户端 (Stdio/HTTP/SSE)
│   │   ├── hooks/          # LLM/工具 Hook
│   │   └── tools/          # 工具基类
│   ├── crewai-tools/       # 80+ 内置工具
│   ├── crewai-core/        # 核心工具 (极简)
│   └── cli/                # CLI 工具
```

**关键特点**：
- 纯 Python 库，无 Web 服务器/API
- 无 LangChain 依赖（完全独立）
- Crews (自主Agent协作) + Flows (事件驱动工作流) 双模型
- AgentExecutor: Plan-and-Act 模式
- 有 a2a 模块（Agent-to-Agent 通信）

### 2.4 Mastra

```
mastra/ (monorepo, 31 packages)
├── packages/
│   ├── core/               # @mastra/core — Agent 运行时 (7086 行主文件)
│   │   ├── agent/          # Agent + SubAgent 接口
│   │   ├── workflows/      # .then()/.branch()/.parallel() 工作流
│   │   ├── tools/          # createTool() + 挂起/恢复
│   │   ├── memory/         # 对话持久化 + 语义召回
│   │   └── hooks/          # ON_GENERATION 等钩子
│   ├── server/             # HTTP 适配器 (Express/Hono/Fastify/Koa)
│   ├── mcp/                # MCP 客户端/服务器
│   ├── memory/             # 独立内存模块
│   ├── rag/                # RAG
│   ├── playground-ui/      # 开发 UI
│   └── client-sdks/        # React SDK (useChat, useAgent)
├── auth/                   # 认证提供者 (Supabase/Firebase/Okta/Clerk)
├── ee/                     # 企业版 License (FGA/RBAC)
└── stores/                 # 存储后端适配
```

**关键特点**：
- TS 原生，架构最现代
- **SubAgent 接口**天然支持外部 Agent 调度
- createTool() 支持 suspend/resume（长任务挂起）
- 无框架绑定（Express/Hono/Fastify 都支持）
- 客户端 SDK 有 useChat/useAgent React hooks
- 40+ LLM Provider
- EE License 限制多租户 FGA/RBAC

### 2.5 Flowise

```
flowise/ (monorepo)
├── packages/
│   ├── server/             # Express.js + oclif CLI
│   │   ├── routes/         # REST API 端点
│   │   ├── database/       # TypeORM (SQLite/MySQL/PostgreSQL)
│   │   ├── services/       # 业务服务
│   │   └── enterprise/     # 企业版 (Workspace/Organization/RBAC)
│   ├── ui/                 # React + MUI v5 + Redux
│   ├── components/         # 100+ LangChain 节点
│   └── agentflow/          # Agent 可视化组件
```

**关键特点**：
- TS 全栈，Apache 2.0
- 100+ 预构建节点（LangChain 生态）
- Chatflow (线性) + Agentflow (LangGraph 状态机)
- Enterprise 已有 Workspace/Organization/RBAC 基础
- 技术栈偏旧（Express v4, Redux, MUI v5）
- 贡献者仅 30 人

## 3. 核心维度评分（1-5）

| 维度 | Dify | Langflow | CrewAI | Mastra | Flowise |
|------|:----:|:--------:|:------:|:------:|:-------:|
| **Agent 核心成熟度** | 4 | 4 | 4 | 5 | 3 |
| **Hook/扩展机制** | 3 | 3 | 4 | 5 | 3 |
| **Skill 支持** | 3 | 3 | 3 | 4 | 3 |
| **MCP 支持** | 5 | 4 | 4 | 4 | 4 |
| **外部Agent调度** | 2 | 2 | 2 | 4 | 2 |
| **架构可Fork性** | 2 | 3 | 4 | 5 | 3 |
| **多租户基础** | 4 | 3 | 1 | 2 | 3 |
| **代码质量** | 3 | 3 | 3 | 4 | 3 |
| **License 友好度** | 3 | 5 | 5 | 3 | 5 |
| **社区/稳定性** | 5 | 5 | 4 | 4 | 3 |

## 4. 改造量估算

| 改造项 | Dify | Langflow | CrewAI | Mastra | Flowise |
|--------|:----:|:--------:|:------:|:------:|:-------:|
| 多租户/项目隔离 | 4-6周 | 2-3周 | 2-3月 | 2-3周 | 2-4周 |
| 外部Agent调度(ACP) | 2-3周 | 2-4周 | 2-4月 | 2-3周 | 1-2周 |
| Agent核心Hook | 1-2周 | 1-2周 | 1-2周 | 3-5天 | 2-3周 |
| Skill系统 | 3-4周 | 2-3周 | 1-2月 | 3-5天 | 2-3周 |
| Web前端(对话UI) | 2-3月 | 1-2月 | 2-3月 | 3-4周 | 3-4周 |
| **总改造量** | **4-6月** | **3-5月** | **8-14月** | **2-3月** | **3-5月** |

## 5. 试用指南

### Dify
```bash
git clone https://github.com/langgenius/dify.git /tmp/dify
cd /tmp/dify/docker
cp .env.example .env
docker compose up -d
# 访问 http://localhost:3000
# 测试：创建 Agent → 添加工具 → 对话 → 观察工具调用
```

### Langflow
```bash
pip install langflow -U
langflow run
# 访问 http://127.0.0.1:7860
# 测试：创建 Flow → 添加 LLM 节点 → 添加 Tool → 运行
```

### CrewAI
```bash
pip install crewai 'crewai[tools]'
crewai create crew test_crew
cd test_crew
# 编辑 src/ 配置 agents.yaml, tasks.yaml
crewai run
# 纯 CLI，无 Web UI
```

### Mastra
```bash
git clone https://github.com/mastra-ai/mastra.git /tmp/mastra
cd /tmp/mastra
corepack enable && pnpm run setup
pnpm build:core && pnpm build:server
# Playground: cd packages/playground-ui && pnpm dev
# 访问 http://localhost:3000
```

### Flowise
```bash
npm install -g flowise
npx flowise start
# 访问 http://localhost:3000
# 测试：创建 Chatflow → 添加节点 → 运行对话
```

## 6. 综合排名

| 排名 | 项目 | 核心优势 | 核心风险 | 推荐理由 |
|------|------|----------|----------|----------|
| 🥇 | **Mastra** | 架构最现代、SubAgent 接口、最小改造量 | EE License 限制、社区较小 | Fork 定制成本最低，TS 原生，天然支持外部调度 |
| 🥈 | **Dify** | 功能最全、企业功能完整、Web UI 成熟 | 体量大(854K LOC)、改造成本高 | 如果需要"开箱即用"的企业平台，Dify 最强 |
| 🥉 | **Langflow** | MIT、Stars 最高、MCP 成熟 | 可视化绑定深、架构可控性待验证 | Python 生态首选，可视化编排能力强 |
| 4 | **Flowise** | TS 全栈、Apache 2.0、改造适中 | 社区小(30贡献者)、技术栈偏旧 | 如果需要 TS + 可视化，是轻量替代 |
| 5 | **CrewAI** | Agent 核心扎实、完全独立 | 无平台层、从零建成本太高 | 适合做 Agent 引擎，不适合做平台底座 |

## 7. pi 项目的定位

pi 不适合做平台底座，但可以作为：
1. **Agent 引擎嵌入**：`agent-core` + `ai` 包嵌入 AIDevOps 的 Agent 运行时
2. **架构参考**：事件驱动扩展系统、Skill 标准的设计参考
3. **技术储备**：如果选 Python 系底座，pi 的 TS Agent 核心可做前端侧补充

---

*报告生成时间: 2026-05-20*
*基于 GitHub 仓库深度代码分析*
