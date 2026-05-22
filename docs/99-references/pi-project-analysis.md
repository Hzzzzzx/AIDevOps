# pi 项目分析

> pi 是用户 workspace 中已有的一个终端 AI 编码 Agent 框架，作为候选底座的评估分析。

## 1. 项目概况

| 项 | 值 |
|---|---|
| 路径 | `/Users/hzz/workspace/pi/` |
| GitHub | `https://github.com/earendil-works/pi.git` |
| 定位 | 可扩展的终端 AI 编码 Agent 框架 |
| 版本 | v0.75.3 |
| License | MIT |
| 技术栈 | TypeScript / Node.js (monorepo) |

## 2. 架构

```
pi/
├── packages/
│   ├── ai/                    # @earendil-works/pi-ai — 统一 LLM API 层
│   │   ├── src/providers/     # 20+ Provider 实现
│   │   ├── models.generated.ts
│   │   └── cli.ts
│   │
│   ├── agent/                 # @earendil-works/pi-agent-core — Agent 运行时核心
│   │   ├── agent.ts           # Agent 类（高级 API）
│   │   ├── agent-loop.ts      # 核心循环
│   │   └── harness/           # 测试工具
│   │
│   ├── coding-agent/          # @earendil-works/pi-coding-agent — CLI 编码 Agent
│   │   ├── cli/               # CLI 入口
│   │   ├── core/              # 核心逻辑 + 内置工具
│   │   ├── modes/             # Interactive/RPC/Print 模式
│   │   └── examples/          # 扩展示例
│   │
│   └── tui/                   # @earendil-works/pi-tui — 终端 UI 框架
│       ├── components/        # Text, Editor, Markdown 等
│       ├── tui.ts
│       └── terminal.ts
```

## 3. 核心能力评估

### Agent 核心 — ✅ 成熟

- `agent-core` 包提供 stateful agent loop
- 事件驱动的工具调用和流式输出
- 会话管理：JSONL 存储、分支、压缩（compaction）

### Skill 系统 — ✅ 完整

- 遵循 agentskills.io 标准
- Skill 以 Markdown 文件定义，按需加载
- 支持自定义触发器、描述、执行逻辑

### MCP — ❌ 故意不支持

> "No MCP. Build CLI tools with READMEs, or build an extension that adds MCP support."

官方设计哲学是"极简核心 + 扩展实现"，MCP 需要自建扩展。

### Web 前端 — ❌ 纯终端

- `web-ui` 包存在但基本是空壳
- 主交互方式是 TUI（终端 UI）

### 扩展机制 — ✅ 事件驱动

扩展是 TypeScript 模块，可以：
- 订阅生命周期事件（session_start, tool_call 等）
- 注册自定义工具
- 添加 slash 命令
- 拦截/修改工具调用
- 访问用户交互（confirm, select, input, notify）

### LLM 支持 — ✅ 20+ Provider

OpenAI, Anthropic, Google, Azure, Bedrock, Mistral, Groq, Cerebras, DeepSeek, xAI, OpenRouter 等

## 4. 设计哲学（刻意排除的能力）

| 排除能力 | 官方说法 | 替代方案 |
|----------|----------|----------|
| MCP | 可通过扩展实现 | 自建扩展 |
| 子 Agent | 可通过扩展或 tmux | 自建 |
| 权限弹窗 | 用容器或自建确认流 | 自建 |
| 计划模式 | 写文件或自建扩展 | 自建 |
| 内置 TODO | 用 TODO.md | 自建 |

"极简核心，最大扩展性"是核心理念。

## 5. 内置工具

read, bash, edit, write, grep, find, ls

## 6. 对 AIDevOps 的适用性分析

### 优势

- `agent-core` 和 `ai` 包架构清晰、质量高，可作为嵌入式 Agent 引擎
- MIT License，无商业限制
- TypeScript 全栈，与前端生态天然契合
- 事件驱动扩展系统设计优雅
- LLM Provider 抽象层完整

### 劣势

- **不是平台**：纯 CLI Agent，无 Web API、无多用户、无项目隔离
- **无 MCP 支持**：需要自建
- **无 Web 前端**：需要从零建设
- **无多 Agent 编排**：需要自建
- **社区较小**：Star 数不高，长期维护存不确定性

## 7. 定位建议

pi **不适合作为 AIDevOps 平台的底座**，但可以作为：

1. **Agent 引擎嵌入**：将 `agent-core` + `ai` 包作为 AIDevOps 的 Agent 运行时
2. **架构参考**：其事件驱动扩展系统、Skill 标准可作为设计参考
3. **技术储备**：如果选了 Python 系的底座（Dify/CrewAI），pi 的 TS Agent 核心可作为前端侧的 Agent SDK 补充

---

*分析时间: 2026-05-20*
