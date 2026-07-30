# PaiAgent 学习路线图 — 按顺序

> 目标：学完 PaiAgent，能回答面试中关于 Agent Harness / LangGraph / RAG / MCP / Skill 的所有问题
> 预计：4 周（每天 1-2 小时）

---

## 学习顺序总览

```
第 1 周    基础架构 + 跑起来（GitHub: 架构图 + README 完善）
第 2 周    核心引擎 DAG + NodeExecutor（GitHub: 技术笔记 + 架构图）
第 3 周    LangGraph4j + ReAct Agent（GitHub: Python LangGraph demo + 技术文章）
第 4 周    Skill / RAG / MCP / Memory（GitHub: Eval 脚本 + README 完整版）
```

---

## 详细周计划

### 第 1 周：让项目跑起来 + 完善 GitHub 门面

**目标**：本地启动 PaiAgent，理解全貌，把 GitHub 仓库 README 补上架构图

**GitHub 产出**：PaiAgent 仓库 README 加入实际的架构图截图 + 启动截图

| 天 | 内容 | 核心文件 | GitHub 产出 |
|----|------|---------|------------|
| Day 1 | 项目全貌浏览，画架构图 | `AGENTS.md` | 画一张架构图（excalidraw 或 draw.io）推至 `docs/architecture.md` |
| Day 2 | 数据库初始化，理解 4 张核心表 | `schema.sql`, `entity/*.java` | `docs/database-schema.md` 表结构笔记 |
| Day 3 | 后端启动 | `application.yml` | 启动成功后截一张 terminal 图，放 repo |
| Day 4 | 前端启动，看到 ReactFlow 画布 | `FlowCanvas.tsx` | 截一张工作流画布图，放 repo |
| Day 5 | 走通一个完整工作流执行 | `WorkflowController.java` | 截 Debug Drawer 执行结果图，放 repo |
| Day 6 | 更新 GitHub 主页 README | - | 用截图 + 架构图更新 README.md |
| Day 7 | 休息 / 复习 | - | - |

### 第 2 周：DAG 引擎 + 节点执行器

**目标**：理解 DAG 引擎原理，写出面试能讲清楚的技术笔记

**GitHub 产出**：技术笔记推至 `docs/`，更新 README 架构描述

| 天 | 内容 | 核心文件 | GitHub 产出 |
|----|------|---------|------------|
| Day 8 | EngineSelector + WorkflowEngine | `EngineSelector.java`, `WorkflowEngine.java` | `docs/engine-architecture.md` |
| Day 9 | DAGParser 逐行读 | `DAGParser.java` | `docs/dag-engine.md`（Kahn + DFS 原理） |
| Day 10 | NodeExecutor 体系一览 | 所有 `executor/impl/*.java` | `docs/node-executors.md` 节点类型表格 |
| Day 11 | LLM 节点 + ChatClientFactory | `LlmNodeExecutor.java`, `ChatClientFactory.java` | 更新 README 中"多 LLM 集成"章节 |
| Day 12 | 条件节点 + 步骤节点 | `ConditionNodeExecutor.java` | `docs/conditional-branching.md` |
| Day 13 | 整理：用面试语言写成笔记 | - | 提交所有 docs/ 更新 |
| Day 14 | 休息 | - | - |

### 第 3 周：LangGraph4j + ReAct Agent（面试核心）

**目标**：理解 LangGraph，在 PaiCLI-Python 里写一个 Python LangGraph demo

**GitHub 产出**：PaiCLI-Python 增加 `examples/langgraph_agent.py` + PaiAgent 更新 docs

| 天 | 内容 | 核心文件 | GitHub 产出 |
|----|------|---------|------------|
| Day 15 | LangGraphWorkflowEngine + WorkflowState | `LangGraphWorkflowEngine.java` | `docs/langgraph-engine.md` |
| Day 16 | GraphBuilder 逐行读 | `GraphBuilder.java` | 同上，补充条件分支说明 |
| Day 17 | StateManager 深入 | `StateManager.java` | `docs/state-management.md` |
| Day 18 | ReActAgentNodeExecutor 逐行读（最重要！） | `ReActAgentNodeExecutor.java` | `docs/react-agent-executor.md` |
| Day 19 | **在 PaiCLI-Python 写 Python LangGraph demo** | 新建文件 `examples/langgraph_agent.py` | PaiCLI-Python 仓库新增 demo |
| Day 20 | 对比 DAG vs LangGraph，写 README 补充 | - | 更新 PaiAgent README，加双引擎对比 |
| Day 21 | 休息 | - | - |

### 第 4 周：Skill / RAG / MCP / Memory + Eval 脚本

**目标**：掌握周边能力，补上评估脚本

**GitHub 产出**：PaiCLI-Python 新增 Eval 测试 + 完善所有 docs

| 天 | 内容 | 核心文件 | GitHub 产出 |
|----|------|---------|------------|
| Day 22 | Skill 系统全套 | `Skill.java`, `SkillLoader.java`, `SkillRegistry.java` | `docs/skill-system.md` + skill SKILL.md 示例 |
| Day 23 | Tool Registry | `AgentTool.java`, `AgentToolRegistry.java` | `docs/tool-registry.md` |
| Day 24 | 知识库 RAG | `KnowledgeBase*.java` | `docs/knowledge-base-rag.md` |
| Day 25 | MCP 配置 + Memory | `McpToolConfig*.java`, `AgentMemoryService.java` | `docs/mcp-integration.md` |
| Day 26 | **在 PaiCLI-Python 写 Eval 脚本** | 新建 `tests/test_eval.py` | PaiCLI-Python 仓库新增 eval 测试 |
| Day 27 | README 最终版：加截图 + 表格 + 架构 | 大更新 | 两个仓库 README 全部完善 |
| Day 28 | 复习总结 | - | 提交最终改动 |

---

## PaiAgent 目录 → 技能对照表

| 目录/文件 | 对应技能 | 面试价值 |
|-----------|---------|---------|
| `engine/dag/DAGParser.java` | 工作流引擎、拓扑排序、环检测 | ★★★★★ |
| `engine/langgraph/*` | LangGraph 状态图、Condition Branch | ★★★★★ |
| `engine/executor/impl/ReActAgentNodeExecutor.java` | Agent Loop 实现 | ★★★★★ |
| `engine/EngineSelector.java` | 工厂模式、多引擎路由 | ★★★★ |
| `engine/skill/*` | Skill 系统、三段式加载 | ★★★★ |
| `engine/agent/*` | Tool Registry 机制 | ★★★★ |
| `engine/llm/ChatClientFactory.java` | 多 LLM 提供商集成 | ★★★★ |
| `service/KnowledgeBaseService.java` | RAG 知识库 | ★★★★ |
| `service/McpToolConfigService.java` | MCP 协议 | ★★★★ |
| `service/AgentMemoryService.java` | 记忆系统 | ★★★ |
| `interceptor/AuthInterceptor.java` | JWT 鉴权 | ★★★ |
| `controller/*.java` | REST API 设计 | ★★ |
| `frontend/components/FlowCanvas.tsx` | 可视化编排 | ★★ |

---

## 面试时 PaiAgent 的 5 个亮点故事

学完以后，你应该能用面试官听得懂的话讲这 5 个故事：

### 故事 1：双引擎架构
"我设计了一个双引擎架构——DAG 引擎适合线性流水线，拓扑排序后按顺序执行；LangGraph 引擎适合 Agent 场景，因为 LLM 需要自己决定下一步调什么工具。EngineSelector 根据工作流的 engineType 动态路由。"

### 故事 2：手写 ReAct Agent 运行时
"在 PaiAgent 的 ReActAgentNodeExecutor 里，我实现了完整的 Agent Loop：调用 LLM → 解析 JSON 决策 → 如果是 tool_call 就去 Registry 查找并执行 → 结果追加到上下文 → 继续循环。每步都有 Trace 记录和 Token 统计。"

### 故事 3：Skill 系统设计
"Skill 系统使用 YAML Frontmatter + Markdown 的 SKILL.md 文件格式。三段式加载：先在 System Prompt 注入 summary（每个 Skill 的摘要），当 Agent 需要时再按需加载 detail 和 reference。这样既让 LLM 知道有什么可用，又不浪费 Token。"

### 故事 4：Tool Registry 机制
"Tool 通过注册表管理，每个 Tool 有 name、description、inputSchema。Agent 执行时从 Registry 获取可用工具列表，组装到 System Prompt 中。新增一个工具只需要实现接口 + 注册，不用改 Agent 逻辑。"

### 故事 5：从 DAG 到 LangGraph 的演进
"为什么从 DAG 演进到 LangGraph？DAG 执行前就确定了拓扑排序，适合固定流水线。但 Agent 场景下，LLM 调完一个工具后，下一步是不可预测的——LangGraph 的 StateGraph + Conditional Edge 天然适合这种运行时决策。"

---

## 学完 PaiAgent 后能补上的面试缺口

| 之前标 ❌ 的技能 | 学完后能回答什么 |
|----------------|---------------|
| LangChain/LangGraph | 能说清楚 LangGraph4j 的 StateGraph 实现 |
| Harness 工程 | PaiAgent 就是 Harness——工具注册、权限、状态管理、Trace |
| Tool Schema 设计 | Tool Registry 的 inputSchema 设计 |
| Agent 循环 | ReActAgentNodeExecutor 的完整实现 |
| Skill 系统 | SKILL.md + 三段式加载 |
| RAG | KnowledgeBase + KnowledgeRetrieveTool |
| MCP | McpToolConfig + SearchInfinityMcpClient |
| 评估 | 可以补 Eval（参考面试成功率提升方案）|

---

## 每天学完后问自己 3 个问题

1. **今天读的核心文件，如果面试官让我用 2 分钟讲清楚，我能说流畅吗？**
2. **这个模块的设计决策是什么？为什么选这种方式而不是另一种？**
3. **这个模块和简历上写的描述能对应上吗？**
