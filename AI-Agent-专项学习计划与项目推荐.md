# AI Agent 专项学习计划与项目推荐

---

## 一、学习路线总览（12 周）

```
第 1-2 周  基础入门    Agent 原理 + 框架选型
第 3-4 周  核心技能    RAG + Tool Calling + MCP
第 5-6 周  Harness 工程  Agent 网关、权限、上下文
第 7-8 周  项目实战    做一个完整可展示的作品
第 9-10 周  Multi-Agent  多 Agent 编排 + 生产部署
第 11-12 周 冲刺优化    面试准备 + 作品打磨 + 开源贡献
```

---

## 二、详细周计划

### 第 1 周：Agent 基础知识

**目标**：理解 Agent 是什么，能说清楚 Agent Loop 的完整流程

- 看 [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) 第 1-3 课
- 看 [Datawhale hello-agents](https://github.com/datawhalechina/hello-agents) 第 1-4 章（概念 + 发展史 + LLM 基础 + 经典范式 ReAct/P&S/Reflection）
- 手写一个 **最小 Agent Loop**（50-100 行 Python）

```python
# 最小 Agent Loop 骨架
def agent_loop(user_input, tools, max_steps=10):
    messages = [{"role": "user", "content": user_input}]
    for step in range(max_steps):
        response = llm(messages, tools=tools)  # 调用 LLM
        if response.stop_reason == "tool_calls":
            for tool_call in response.tool_calls:
                result = execute_tool(tool_call)  # 执行工具
                messages.append({"role": "tool", ...})
        else:
            return response.content
    return "Max steps reached"
```

**交付物**：一个能调函数/搜索的极简 Agent demo，README 说明架构

---

### 第 2 周：框架选型与上手

**目标**：选一个主攻框架，跑通官方示例

**主攻选型建议**：

| 框架 | 适合场景 | 理由 |
|---|---|---|
| **LangChain/LangGraph** | 通用 Agent 开发 | 生态最大，JD 出现率最高，推荐主攻 |
| **OpenAI Agents SDK** | 快速原型 | 官方出品，代码简洁，适合入门理解 |
| **AutoGen** | 企业级多 Agent | 微软背书，国内企业采用率高 |

**行动**：

- 跑通 LangChain 官方 Quickstart
- 跑通 LangGraph 的 Agent 示例
- 对比两种实现方式的差异（Chain vs Graph）

**交付物**：用 LangGraph 实现一个带 Tool Calling 的问答 Agent

---

### 第 3 周：RAG 流水线

**目标**：掌握 RAG 全流程，能独立搭建知识库问答系统

**学习内容**：

- 文档解析（PDF/Markdown/HTML）
- Chunk 策略（Fixed-size, Semantic, Agentic）
- Embedding 模型选型（text-embedding-3-small, bge-m3, m3e）
- 向量数据库（Chroma 上手最简单，Milvus 最工业级）
- Retrieval 策略（相似度搜索、MMR、Hybrid Search）
- Reranker 排序
- 带引用的 Answer Generation

**推荐项目**：**Paper Agent**（论文阅读助手）

```
用户输入论文链接/PDF → 解析 → Chunk → Embed → 存储 → 
用户提问 → Retrieve → Rerank → LLM 生成带引用的回答
```

**交付物**：一个能上传 PDF 并回答问题的 RAG Agent

---

### 第 4 周：MCP 协议

**目标**：理解 MCP 协议，能自己写 MCP Server

**学习内容**：

- 看 [microsoft/mcp-for-beginners](https://github.com/microsoft/mcp-for-beginners) Module 0-3
- 理解 MCP 核心概念：Resources, Tools, Prompts, Sampling
- 理解 stdio 传输 vs HTTP 传输
- 写第一个 MCP Server（如：天气查询、文件操作、数据库查询）

**交付物**：一个功能性 MCP Server（建议做 **PostgreSQL MCP Server** 或 **文件系统 MCP Server**），能被 Claude/Cline 调用

---

### 第 5 周：Agent Harness 工程

**目标**：理解 Agent 生产化的基础设施，这是 **#1 招聘缺口**

**学习内容**：

- 工具注册与发现（Tool Registry）
- 权限门控（Permission Gate）
- 上下文管理（Context Window 控制、压缩策略）
- 会话存储（Session Store）
- 链路追踪（Trace Logging）
- 错误恢复与重试
- 沙箱执行（Sandbox）

**推荐项目**：**Tiny Agent Gateway**

```
输入 → 认证/权限 → 上下文组装 → Agent Loop → 工具执行（沙箱）→ 
Trace 记录 → 输出 → 会话存储
```

参考 read-pdf 等实际 Skill 的实现结构，理解 `SKILL.md` + 工具的注册模式。

**交付物**：一个小型 Agent Gateway demo，展示工具注册 + 权限 + trace 三大模块

---

### 第 6 周：Context Engineering + 记忆系统

**目标**：掌握上下文工程这一新兴关键能力

**学习内容**：

- [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) 第 12 课 Context Engineering
- 上下文压缩策略：总结、丢弃、优先级排序
- 记忆系统：短期（窗口）、中期（摘要）、长期（向量/知识图谱）
- 多轮对话中的上下文管理

**行动**：

- 对比不同 Context 管理策略的效果
- 实现一个带记忆管理的 Agent

**交付物**：对 Harness 项目增加上下文压缩 + 记忆管理模块

---

### 第 7-8 周：完整项目实战（核心阶段）

选择一个完整项目认真完成，这是简历上的 **关键作品**。推荐以下方向：

#### 项目 A：编码 Agent（最通吃，推荐首选）

构建一个类 Claude Code 的编码 Agent，能读写文件、执行命令、在代码库中完成任务。

```
技术栈：Python + LangGraph + MCP File System Server
功能：
  - 文件读写（单个 + 批量）
  - Shell 命令执行（沙箱）
  - 代码搜索（Grep/Glob）
  - 文件编辑（Find & Replace）
  - 项目上下文理解
```

**参考**：[shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) — 7000 行的 Nano Claude Code

#### 项目 B：Deep Research Agent

复现类 ChatGPT Deep Research 的深度研究 Agent。

```
技术栈：Python + LangGraph + Tavily/Brave Search + MCP
流程：
  用户提问 → 规划子问题 → 并行搜索 → 阅读 → 
  整合 → 发现矛盾 → 再次搜索 → 输出结构化报告
```

**参考**：hello-agents 第 14 章 DeepResearch Agent 复现

#### 项目 C：Multi-Agent 协作系统

```
技术栈：LangGraph / CrewAI + MCP
Agent 角色：
  - Planner：分解任务
  - Researcher：搜索与收集信息
  - Writer：撰写内容
  - Reviewer：审查与修订
```

**交付物**：GitHub 开源项目，包含 README、架构图、部署文档、demo 视频链接

---

### 第 9-10 周：Multi-Agent + 生产部署

**学习内容**：

- [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) 第 8 课 Multi-Agent
- [hello-agents](https://github.com/datawhalechina/hello-agents) 第 11 章 评估体系
- Docker 容器化 Agent
- CI/CD for Agents
- 监控与可观测性（LangSmith、LangFuse）
- 成本优化（缓存、模型路由、Token 管理）

**交付物**：将第 7-8 周的项目容器化，加 CI/CD + 监控

---

### 第 11-12 周：面试准备 + 开源贡献

**面试准备**：

- 刷 [AgentGuide 面试题库](https://github.com/adongwanai/AgentGuide)（1000+ 题）
- 准备以下常见面试题：
  - Agent Loop 的实现原理
  - LangGraph 的状态管理机制
  - RAG 的 Chunk 策略选择
  - Context Window 满了怎么办
  - MCP 和 Function Calling 的区别
  - Multi-Agent 的通信与协调方案
  - Agent 的安全性（Prompt Injection、工具权限）
- 准备项目演示：架构设计、关键难点、技术选型理由

**开源贡献**：

- 给 LangChain / LangGraph 提 PR（即使是文档改进）
- 发布自己做的 MCP Server 到 npm/pip
- 在 GitHub Discussions 中活跃参与 AI Agent 话题

---

## 三、项目推荐总表

| 项目 | 周期 | 难度 | 技术栈 | 展示价值 |
|---|---|---|---|---|
| **编码 Agent** | 2 周 | 高 | Python + LangGraph + MCP | ★★★★★ |
| **Paper Agent** | 1 周 | 中 | Python + RAG + Chroma | ★★★★ |
| **Travel Agent** | 2 周 | 高 | Python + MCP + Multi-Agent | ★★★★ |
| **Deep Research Agent** | 2 周 | 高 | Python + LangGraph + Search | ★★★★★ |
| **MCP Server** | 1 周 | 中 | Python/TS + MCP SDK | ★★★ |
| **Agent Gateway/Harness** | 2 周 | 高 | Python + 权限 + Trace | ★★★★★ |
| **Web Agent** | 2 周 | 高 | Playwright + LLM | ★★★★ |
| **赛博小镇** | 3 周 | 高 | Multi-Agent + Game | ★★★ |
| **Tiny Claude Code** | 2 周 | 高 | Agent Loop + Tool Use | ★★★★★ |
| **Multi-Agent 写作系统** | 1 周 | 中 | CrewAI/LangGraph | ★★★★ |

---

## 四、学习资源索引

### 教程课程

| 资源 | 适合阶段 | 链接 |
|---|---|---|
| Datawhale hello-agents | 入门→高级 | [github.com/datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents) |
| Microsoft AI Agents for Beginners | 入门→中级 | [github.com/microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) |
| Microsoft MCP for Beginners | 中级 | [github.com/microsoft/mcp-for-beginners](https://github.com/microsoft/mcp-for-beginners) |
| AgentGuide（求职向） | 全阶段 | [github.com/adongwanai/AgentGuide](https://github.com/adongwanai/AgentGuide) |
| Microsoft LangChain for Beginners | 中级 | Microsoft Learn |
| shareAI-lab/learn-claude-code | 高级 | [github.com/shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) |

### 推荐框架优先级

```
首选：LangChain + LangGraph（生态最大，JD 出现率最高）
次选：OpenAI Agents SDK（官方出品，快速上手）
补充：AutoGen（企业级）、CrewAI（多 Agent）、Pydantic AI（类型安全）
关注：Mastra（TypeScript 新星）
```

### 中文社区资源

- **知乎**：搜索 "AI Agent 开发实战"、"LangGraph 教程"
- **B站**：Datawhale hello-agents 视频讲解
- **GitHub**：AgentGuide、hello-agents 的 discussions

---

## 五、关键建议

1. **先做后学** — 不要花太多时间看文档，第 1 周就要写代码跑 Agent Loop
2. **作品导向** — 简历上 1 个高质量编码 Agent > 3 个入门 demo
3. **开源可见** — 项目发 GitHub，README 写清楚架构和技术选型理由
4. **理解深一层** — 面试考的不是会不会用 LangChain，是懂不懂 Agent 原理
5. **关注 Context Engineering** — 这是 2026 年最热的新技能点
6. **MCP 是必选项** — 面试中 MCP 相关问题的出现频率在快速上升
7. **英语能力** — 大部分高质量文档和社区讨论是英文的
8. **保持更新** — AI Agent 领域变化极快，每周看一次 GitHub Trending

---

*计划编制于 2025-07-20，基于最新市场数据*
