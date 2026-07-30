# AI Agent 学习计划（时间约束优化版）

> 假设：每日高质量深度专注时间 **240 分钟（4 小时）**
> 单次持续专注窗口：**10-15 分钟**
> 总周期：**12 周 = 60 个学习日（周末双休）**

---

## 一、时间模型

### 每日时间分配

```
上午 2 小时（最佳状态）：新概念学习 + 编码实践
下午 1 小时：复习 + 阅读 + 整理笔记
晚上 1 小时：社区互动 + 轻量任务
```

### 专注节奏（每个小时）

```
15 min  深度专注 Sprint 1
 5 min   休息（离开屏幕、走动、喝水）
15 min  深度专注 Sprint 2
 5 min   休息
15 min  深度专注 Sprint 3
 5 min   休息
```

每小时包含 **3 个专注 sprint**，每日 4 小时 = **12 个专注 sprint**。

### 周节奏

| 日 | 类型 | 学习量 |
|---|---|---|
| 周一至周五 | 学习日 | 4 小时 / 天 |
| 周六 | 项目日 | 专注做项目 |
| 周日 | 休息/补漏 | 复盘、整理、放松 |

12 周 ≈ 60 个学习日 + 12 个项目日 = **72 个有效日**

---

## 二、12 周逐日计划

---

### 第 1 周：Agent 原理 + 最小实现

**目标**：理解 Agent Loop，手写第一个 Agent

#### Day 1 — 理解 Agent 核心概念

| Sprint | 内容 | 时长 |
|---|---|---|
| 1 | 读 Datawhale hello-agents 第 1-2 章：Agent 定义、发展史、能力边界 | 15 min |
| 2 | 画一张 Agent 架构草图：Input → Plan → Tool Call → Observe → Loop | 15 min |
| 3 | 理解 ReAct 模式：Reasoning + Acting 交替的核心思想 | 15 min |
| 4 | 写一段伪代码描述 Agent Loop 流程 | 15 min |
| 5 | 读 LangChain 官方文档的 Agent 概念介绍 | 15 min |
| 6 | 对比 Agent vs Chain vs Graph 的区别 | 15 min |
| *午休* | | |
| 7 | 观看一段 15 分钟的 Agent 原理讲解视频 | 15 min |
| 8 | 读 OpenAI Function Calling 文档 | 15 min |
| 9 | 跑通 OpenAI API 的 Function Calling 示例 | 15 min |
| 10 | 理解 Tool Schema 定义格式 | 15 min |
| 11 | 整理笔记：Agent Loop 的 5 个关键环节 | 15 min |
| 12 | 写一篇 200 字的学习总结 | 15 min |

#### Day 2 — 手写最小 Agent Loop

| Sprint | 内容 | 时长 |
|---|---|---|
| 1 | 搭建 Python 环境，准备 OpenAI/DeepSeek API Key | 15 min |
| 2 | 写 LLM 调用函数：messages → response | 15 min |
| 3 | 写 Tool Call 解析函数：从 response 中提取 tool_call | 15 min |
| 4 | 写一个简单工具：`calculate(expr)` | 15 min |
| *休息* | | |
| 5 | 写 Agent Loop 主循环：for step in range(max_steps) | 15 min |
| 6 | 处理"无 tool_call"终止条件 | 15 min |
| 7 | 测试：让 Agent 计算 `(15+7)*3-8/2` | 15 min |
| 8 | 调试：处理解析错误、重试逻辑 | 15 min |
| *午休* | | |
| 9 | 加第二个工具：`search_wikipedia(query)` 模拟 | 15 min |
| 10 | 加第三个工具：`get_current_time()` | 15 min |
| 11 | 加最大步数限制和错误处理 | 15 min |
| 12 | 测试多工具场景，记录失败案例 | 15 min |

**Day 1-2 交付物**：`agent_loop.py` — 最小 Agent Loop，支持 3+ 工具

#### Day 3 — 跑通 LangChain

| Sprint | 内容 |
|---|---|
| 1-2 | LangChain 环境搭建 + 基础 LLM 调用 |
| 3-4 | LangChain Tool 定义方式 |
| 5-6 | LangChain AgentExecutor 跑通 |
| 7-8 | 对比：手写的 Agent vs LangChain 的差异 |
| 9-10 | 理解 AgentType (ZERO_SHOT_REACT, OPENAI_FUNCTIONS) |
| 11-12 | 用 LangChain 重写 Day 2 的 Agent |

#### Day 4 — 跑通 LangGraph

| Sprint | 内容 |
|---|---|
| 1-2 | LangGraph 核心概念：StateGraph, Node, Edge |
| 3-4 | 理解 State 管理与 Reducer |
| 5-6 | 用 LangGraph 实现 Agent（ToolNode 模式） |
| 7-8 | 理解 Conditional Edge |
| 9-10 | 对比 LangChain Agent vs LangGraph Agent |
| 11-12 | 在 LangGraph 中添加错误处理和重试 |

#### Day 5 — 框架对比与总结

| Sprint | 内容 |
|---|---|
| 1-2 | 读 hello-agents 第 4-5 章：经典范式 + 低代码平台 |
| 3-4 | 了解 AutoGen 的基本概念 |
| 5-6 | 了解 CrewAI 的基本概念 |
| 7-8 | 制作框架对比表格（适用场景/复杂度/生态） |
| 9-10 | 选定主攻方向（推荐 LangGraph） |
| 11-12 | 撰写本周学习总结 README |

**第 1 周交付物**：
- `agent_loop.py` — 手写最小 Agent（50-100 行）
- `langgraph_agent.py` — LangGraph 版 Agent
- `README.md` — 架构说明 + Agent Loop 原理

---

### 第 2 周：工具调用 + OpenAI Agents SDK

#### Day 6-7 — Tool 设计与注册

| 学习点 |
|---|
| Tool Schema 设计规范 |
| 参数类型：string, number, enum, array, object |
| Required vs Optional 参数 |
| Tool 描述与枚举值对 LLM 调用准确性的影响 |
| Tool 注册模式：注册表 + 查找 + 执行 |

**项目**：实现一个 Tool Registry 模块

工具清单建议：
- `read_file(path)` / `write_file(path, content)`
- `execute_command(command)`
- `web_search(query)`
- `list_directory(path)`

#### Day 8-9 — OpenAI Agents SDK

| 学习点 |
|---|
| Agent 定义方式 |
| Handoff 机制 |
| Guardrails |
| Tracing |

**对比**：OpenAI Agents SDK vs LangGraph，从代码量、灵活性、生态角度对比

#### Day 10 — 轻量项目：命令行 Agent

做一个能在命令行运行的 Agent：

```
$ python my_agent.py "帮我找到当前目录下最大的文件"
→ 列出文件 → 排序 → 输出结果

$ python my_agent.py "把 src/ 下的 py 文件行数统计一下"
→ 扫描 → 统计 → 输出表格
```

---

### 第 3 周：RAG 流水线

#### Day 11-13 — RAG 核心组件

| 学习点 | 实践 |
|---|---|
| 文档解析 | 用 LangChain 的 PDF 解析器 |
| Chunk 策略 | 实现 3 种策略对比（固定大小/语义/递归） |
| Embedding | 调通 text-embedding-3-small |
| 向量存储 | 上手 Chroma（20 分钟即可跑通） |
| Retrieval | 相似度搜索 + MMR |
| Reranker | 理解 reranker 的作用 |

#### Day 14-15 — 项目：Paper Agent

构建论文阅读助手：

```
输入 PDF → 解析 → Chunk → 向量化 → 存储 →
提问 → 检索 → Rerank → 带引用回答
```

**重点**：展示你对 Chunk 策略选择的理解，为什么选这种策略而不是另一种。

---

### 第 4 周：MCP 协议

#### Day 16-18 — MCP 原理与上手

| 学习点 |
|---|
| MCP 核心概念：Resources, Tools, Prompts, Sampling |
| stdio 传输 vs Streamable HTTP |
| MCP Server 的目录结构 |
| MCP Client 的连接方式 |

**实践**：跑通 [microsoft/mcp-for-beginners](https://github.com/microsoft/mcp-for-beginners) 的 Module 3

#### Day 19-20 — 写第一个 MCP Server

实现一个 **文件系统 MCP Server**：

```
Tools:
  - read_file(path): 读取文件
  - write_file(path, content): 写入文件
  - list_directory(path): 列出目录
  - grep(pattern, path): 搜索文件内容
Resources:
  - file://{path}: 文件内容资源
```

**交付物**：能在 Claude Desktop / Cline 中调用的 MCP Server

---

### 第 5 周：Agent Harness 工程

> 这部分是 **#1 招聘缺口**，建议花最多时间

#### Day 21-22 — 工具注册与权限

| 学习点 |
|---|
| Tool Registry 设计 |
| 权限门控（读/写/执行 三级） |
| 白名单 vs 黑名单 |
| 参数校验与消毒 |

#### Day 23-24 — 上下文管理

| 学习点 |
|---|
| Context Window 估算 |
| Token 计数 |
| 上下文压缩策略（丢弃、摘要、优先级排序） |
| 记忆管理（短期 + 中期 + 长期） |

#### Day 25 — 链路追踪

| 学习点 |
|---|
| 每一步 Agent 的输入输出记录 |
| Tool Call 耗时统计 |
| Token 消耗追踪 |
| 调试信息的结构化输出 |

---

### 第 6 周：Context Engineering + 记忆系统

#### Day 26-28 — Context Engineering 深度理解

| 学习点 |
|---|
| 信息注入策略 |
| System Prompt 的结构化设计 |
| 动态上下文裁剪 |
| 长对话的退化问题与缓解方案 |

#### Day 29-30 — 整合到 Harness

将第 5 周的 Harness 加入上下文压缩 + 记忆管理模块。

---

### 第 7-8 周：完整项目（核心交付）

选择一个项目投入 **2 周项目时间**：

#### 选项 A：编码 Agent（★★★★★ 推荐）

**特点**：最贴近实际工作场景，面试最能打动面试官

**功能清单**：
- [x] 文件读写（单个 + 批量）
- [x] Shell 命令执行（权限控制 + 沙箱）
- [x] 代码搜索（Grep/Glob）
- [x] 文件编辑（Find & Replace）
- [x] 项目上下文理解
- [x] 错误自动修复
- [x] Trace 日志

**时间安排**：

| 日 | 内容 |
|---|---|
| Day 31 | 搭建项目骨架 + 文件读写模块 |
| Day 32 | Shell 执行 + 安全沙箱 |
| Day 33 | 代码搜索模块 |
| Day 34 | 文件编辑模块 |
| Day 35 | 上下文理解模块 |
| Day 36 (周六) | 联调 + 修复边界情况 |
| Day 37 | 休息 |
| Day 38 | Agent Loop 集成 |
| Day 39 | 错误处理 + 重试逻辑 |
| Day 40 | Trace 日志 + Debug 模式 |
| Day 41 | README 文档 + 架构图 |
| Day 42 | demo 脚本 + 测试用例 |
| Day 43 (周六) | 最终打磨 + 发布 GitHub |
| Day 44 | 休息 |

#### 选项 B：Deep Research Agent

**特点**：展示规划能力 + 搜索整合能力

**核心功能**：子问题分解 → 并行搜索 → 阅读总结 → 交叉验证 → 产出结构化报告

#### 选项 C：Multi-Agent 写作系统

**特点**：展示多 Agent 编排能力

**Agent 角色**：Planner → Researcher → Writer → Reviewer

---

### 第 9 周：Multi-Agent 编排

#### Day 45-47 — Multi-Agent 模式

| 学习点 |
|---|
| 任务分解模式 |
| 通信模式（直接调用 vs 消息队列） |
| Agent 角色定义 |
| 结果聚合策略 |
| 冲突解决机制 |

#### Day 48-49 — 生产部署基础

| 学习点 |
|---|
| Docker 容器化 Agent |
| 环境变量管理 |
| 日志收集 |
| 简单的 CI/CD 流水线 |

---

### 第 10 周：生产级能力

#### Day 50-52 — 生产化

| 学习点 |
|---|
| 成本优化（缓存、模型路由、Token 池化） |
| 监控与告警（LangFuse、LangSmith） |
| 性能评估（Eval 体系） |
| A/B 测试 |

#### Day 53-54 — 安全

| 学习点 |
|---|
| Prompt Injection 防御 |
| 工具权限最小化原则 |
| 输入消毒与输出审查 |
| 审计日志 |
| 速率限制 |

---

### 第 11 周：面试准备

#### Day 55-57 — 刷题

| 主题 | 关键问题 |
|---|---|
| Agent 原理 | Agent Loop 怎么实现的？ReAct 是什么？ |
| LangGraph | StateGraph vs MessageGraph？Conditional Edge？ |
| RAG | Chunk 策略怎么选？Hybrid Search 原理？ |
| MCP | MCP 和 Function Calling 区别？Resources 是什么？ |
| Context | Context 满了怎么办？压缩策略有哪些？ |
| Multi-Agent | Agent 间如何通信？任务怎么分解？ |
| 安全 | 怎么防止 Prompt Injection？ |

#### Day 58-59 — 作品包装

| 任务 |
|---|
| 完善 GitHub README（架构图 + 技术选型理由 + 运行步骤） |
| 准备 5 分钟 demo 脚本 |
| 准备面试讲故事的脚本（难点 → 解决 → 效果） |
| 准备技术选型的对比理由 |

#### Day 60 — 模拟面试

---

### 第 12 周：开源贡献 + 持续更新

| 任务 |
|---|
| 为用过的框架提文档改进 PR |
| 发布 MCP Server 到公开注册表 |
| 在 GitHub Discussions 中参与技术讨论 |
| 写一篇技术博客总结项目经验 |
| 关注 GitHub Trending 保持更新 |

---

## 三、各阶段精力分配

```
第 1-2 周   学习 70% / 编码 30%    打基础
第 3-4 周   学习 40% / 编码 60%    边学边做
第 5-6 周   学习 30% / 编码 70%    实践为主
第 7-8 周   学习 10% / 编码 90%    全力做项目（核心交付）
第 9-10 周  学习 40% / 编码 60%    补生产级能力
第 11 周    面试 60% / 打磨 40%    冲刺
第 12 周    开源贡献 50% / 学习 50% 收官
```

---

## 四、关键原则

### 每 15 分钟一个 Sprint

每个 Sprint 有明确的起点和终点，避免"学一会儿就分心"的陷阱。

**Sprint 模板**：
```
[00:00-00:15] Sprint：___________________
[00:15-00:20] 休息
[00:20-00:35] Sprint：___________________
```

### 每个 Sprint 只做一件事

| 正确 ❌ | 错误 ❌ |
|---|---|
| "跑通 LangChain Quickstart" | "学习 LangChain" |
| "写出 Tool Schema 定义" | "做 Agent" |
| "测试 Chunk 策略 A vs B" | "搞 RAG" |

### 遇到卡住时的原则

1. Sprint 内卡住 → 标记问题，Sprint 结束后统一查
2. 卡住 3 个 Sprint → 切换任务，明天继续
3. 连续多天卡同一个点 → 降低难度，换个方向
4. 不要在一个 15 分钟 Sprint 里同时做"学"和"做"—— 要么看文档，要么写代码

### 如何维持 4 小时高质量专注

- **早上优先**：最重要的学习任务放在前 2 小时
- **环境隔离**：学习时手机静音、通知关闭
- **外部化记忆**：笔记要记在纸上/文档里，不要记在脑子里
- **每个 Sprint 后问自己**："刚才这 15 分钟我学到了什么？"
- **晚上复盘**：睡前花 5 分钟回顾今天的 12 个 Sprint，标记效果最好的 3 个

---

## 五、总投入统计

| 项目 | 数值 |
|---|---|
| 总学习天数 | 72 天 |
| 每日深度专注时间 | 4 小时（240 分钟） |
| 每日专注 Sprint 数 | 12 个（每个 15 分钟） |
| 总专注时间 | **288 小时** |
| 总专注 Sprint 数 | **864 个** |
| 完整项目数 | **至少 1 个**（第 7-8 周） |
| 小型实践数 | **6-8 个**（每周 1 个） |
| 休息日 | 每周日 + 每隔一周的周六 |

---

*计划编制于 2025-07-20*
*以 4 小时/天深度专注、15 分钟/sprint 为基本单元*
