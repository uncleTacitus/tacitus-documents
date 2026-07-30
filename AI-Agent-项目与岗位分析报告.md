# AI Agent 项目与岗位分析报告

> 生成时间：2025-07-20
> 数据来源：GitHub Trending、Boss直聘、智联招聘、行业求职指南

---

## 一、GitHub 近期热门 AI 项目（Top 框架类）

| 项目 | ⭐ Stars | 语言 | 定位 |
|---|---|---|---|
| **langchain-ai/langchain** | 142k | Python | Agent 工程平台 |
| **FoundationAgents/MetaGPT** | 69k | Python | 多智能体软件公司模拟 |
| **microsoft/autogen** | 60k | Python | 微软 Agent 编程框架 |
| **crewAIInc/crewAI** | 56k | Python | 角色扮演多 Agent 编排 |
| **langchain-ai/langgraph** | 38k | Python | 弹性 Agent 构建 |
| **openai/openai-agents-python** | 28k | Python | 官方轻量多 Agent SDK |
| **mastra-ai/mastra** | 26k | TypeScript | 现代 TypeScript Agent 框架 |
| **pydantic/pydantic-ai** | 19k | Python | Pydantic 风格 Agent 框架 |
| **elizaOS/eliza** | 19k | TypeScript | 开源 Agent 操作系统 |
| **camel-ai/camel** | 17k | Python | 首个多 Agent 框架 |

**超新星级别项目：**

| 项目 | ⭐ Stars | 说明 |
|---|---|---|
| **obra/superpowers** | 258k | Agent 技能框架与软件开发方法论 |
| **browser-use/browser-use** | 106k | 让 AI Agent 操控浏览器 |
| **firecrawl/firecrawl** | 153k | AI Agent 网页抓取 API |
| **microsoft/ai-agents-for-beginners** | 70k | 微软 18 课免费 Agent 教程 |

**国内相关资源：**

| 资源 | ⭐ Stars | 说明 |
|---|---|---|
| **adongwanai/AgentGuide** | 7k | AI Agent 岗位求职系统指南 |
| **liyupi/ai-guide** | 17k | AI 资源导航 + Vibe Coding 教程 |
| **ageerle/ruoyi-ai** | 5.5k | 企业级 AI 框架（Java），支持多 LLM + 可视化工作流 |

---

## 二、AI Agent 岗位 JD 普遍要求

根据 Boss直聘 / 智联招聘 10k+ 岗位的数据梳理，技能要求分为 **4 个层级**：

### L1 — 基础层（必备）

- **Python**（必须，占 90%+ 岗位）或 **TypeScript**（快速增长）
- **LLM API 调用**：OpenAI、Claude、DeepSeek、GLM、Qwen
- **结构化输出（JSON Mode）& Function / Tool Calling**
- **Prompt Engineering** → 已演进为 **Context Engineering**

### L2 — 核心开发能力

- **LangChain / LangGraph** — Agent 工作流与状态编排
- **RAG 流水线**：chunk → embed → retrieve → answer with citations
- **向量数据库**：Chroma、Milvus、Pinecone、Weaviate
- **Tool 定义与 Schema 设计**
- **Agent 循环**：observe → think → act → observe
- **记忆系统**：短期上下文 / 会话记忆 / 长期记忆
- **MCP 协议**（Model Context Protocol）— 连接工具与数据源
- **多 Agent 编排**：planner / executor / reviewer / critic 模式

### L3 — 进阶能力（区分度）

- **Agent Harness 工程** — 工具注册、权限门控、上下文压缩、链路追踪
- **Context Engineering** — 控制信息如何进入模型（不再是写提示词）
- **Skills / MCP / A2A / ACP 协议**
- **评估与可观测性**：evals、tracing、安全边界
- **Browser / Computer-Use Agent**（Playwright、browser-use）
- **Claude Code / Codex 式编码 Agent**
- **微调与强化学习**（SFT、RL）

### L4 — 生产级能力

- Docker & 部署
- CI/CD for Agents
- 权限与安全系统
- 成本优化与延迟管理
- 监控与调试（LangSmith、LangFuse）

### 常见 JD 关键词

```
Python、LangChain、LangGraph、RAG、大模型调优、Agent 架构设计、
MCP、Multi-Agent、向量数据库、Prompt Engineering、Docker
```

---

## 三、职业方向分类

国内 AI Agent 岗位主要分两条路线：

| 方向 | 侧重 | 技能要求 |
|---|---|---|
| **算法工程师**（模型算法） | 模型训练、微调、RL | 深度学习、ML/DL 理论、模型微调、强化学习 |
| **开发工程师**（应用开发） | Agent 系统搭建 | LangChain、RAG、多 Agent 编排、工程落地 |

**通吃策略**：两条路线的核心技能都要覆盖。

### 薪资参考

| 级别 | 月薪范围（RMB） |
|---|---|
| 入门 | 15k - 30k |
| 中级 | 30k - 60k |
| 高级 | 60k+ |

热门城市：北京、上海、深圳、杭州

---

## 四、最"通吃"的项目类型（按优先级）

| 优先级 | 项目类型 | 展示什么能力 |
|---|---|---|
| 1 | **编码 Agent**（类似 Claude Code） | 代码库读写、Shell 执行、文件编辑、权限控制 |
| 2 | **Agent Harness 工程** | 工具注册、权限门控、状态管理、trace 日志 |
| 3 | **Context Engineering 系统** | 信息如何进入模型、压缩策略 |
| 4 | **Skills / MCP 系统** | 可复用的 SKILL.md 工作流 |
| 5 | **Multi-Agent 协作系统** | Planner → Researcher → Writer → Reviewer 管线 |
| 6 | **Browser / Computer-Use Agent** | 网页自动化 + 安全边界 |

### 具体项目创意

- **Paper Agent** — 自动论文阅读与 RAG 摘要
- **Travel Agent** — 多步旅行规划 + 工具集成
- **Web Agent** — 基于浏览器的任务自动化
- **TinyClaw** — 自改进的 RAG-to-Skill 框架
- **OpenClaw / claw0** — 带记忆与心跳的长运行编码 Agent

---

## 五、推荐构建路径（8-12 周）

```
Week 1    从零手写最小 Agent Loop（50-150 行），再移植到 LangChain
Week 2    构建 RAG + 记忆系统的研究助手 Agent
Week 3-4  构建 Harness Demo：工具注册、权限门控、会话存储、trace
Week 5-6  实现可复用 Skill + MCP Server，开源发布
Week 7-8  构建 Multi-Agent 管线，聚焦任务分解与上下文隔离
```

---

## 六、核心结论

1. **Context Engineering > Prompt Engineering** — 市场已发生转变，能控制信息如何进入模型比会写提示词更值钱
2. **Harness Engineering 是 #1 招聘缺口** — 理解模型周边的工具、权限、状态、反馈基础设施的人才稀缺
3. **最能通吃的项目组合 = 编码 Agent + Harness 工程 + MCP/Skills 系统**
4. **Python 为主，TypeScript 为辅** — 90%+ 框架用 Python，TypeScript 在快速增长
5. **中文大模型厂商集成经验加分** — DeepSeek、GLM、Qwen 在国内 JD 中出现频率很高
6. **评估与安全是分水岭** — 没有 eval、没有 trace、没有权限边界的 Agent 只是 demo
7. **Skills/SOP 自动化是最新热门** — 能将领域工作流编码为可复用 Skill 的能力在 2025-2026 年被高度认可

---

*报告完*
