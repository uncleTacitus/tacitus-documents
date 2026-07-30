# 曾文 - AI Agent 后端工程师

17398220931 (同微信) | 17398220931@163.com | https://github.com/uncleTacitus

---

## 专业技能

| 分类 | 技能 |
|------|------|
| **AI Agent** | ReAct Agent Loop、Plan-and-Execute、Multi-Agent 协作（Planner/Worker/Reviewer）、MCP Client/Server、Tool Calling & Registry、Skill 系统、RAG（Embedding/语义检索/代码索引）、记忆系统（短期/长期/快照恢复）、HITL 人工确认、Runtime API、快照回滚 |
| **编程语言** | Python（FastAPI、SQLAlchemy 2.0 async、Pydantic v2、asyncio）、Java（Spring Boot 3.4、MyBatis） |
| **后端框架** | FastAPI 0.115、Spring Boot 3.4、Spring Cloud、Spring AI 1.0、MyBatis-Plus |
| **数据库** | PostgreSQL 16（asyncpg）、MySQL 8.0、SQLite、Elasticsearch、Redis 7 |
| **中间件** | RabbitMQ、Kafka、RocketMQ、Redis（持久化/集群/分布式锁/Lua） |
| **AI/LLM** | LangChain/LangGraph、LangGraph4j、OpenAI / DeepSeek / Qwen / ZhiPu / GLM 多提供商、多模态（图片输入）、Context Engineering、Prompt 分层架构 |
| **分布式** | Spring Cloud Gateway、Nacos、Seata AT、Dubbo、ZooKeeper |
| **工具** | Docker、Git、Alembic、FastExcel、Linux、JWT、微信支付/商户、腾讯云 COS/SMS、MinIO |

---

## 工作经历

### 荆州大宅门装饰 | 后端开发工程师（Python/FastAPI） 2025.06 - 至今

负责家装行业分销与线索运营平台「大宅门推客系统」的后端架构设计与开发，覆盖多角色权限体系、线索全生命周期状态机、奖励佣金链路与微信支付；同时设计并落地企微 AI 机器人的数据图表直查系统。

### 博士眼镜(海南)投资控股有限公司 | 运营数据分析与系统流程专员 2023.10 - 2025.08

独立负责海口区域全链路运营管理，从销售接单、定制加工、质检到门店交付的端到端流程优化。管理全市多门店库存，主导采购、仓储、调拨及周期盘点。通过优化加工排程与匹配规则，将平均订单处理时长缩短约 15%。培养了深刻的业务洞察力与系统流程思维。

### 武汉尚高软件有限公司 | Java 后端开发（实习） 2023.02 - 2023.05

独立完成客户管理、订单处理等核心模块的后端开发，参与需求评审、编码、测试及 Bug 修复全流程。

---

## 项目经历

### PaiCLI-Python — 终端 AI Agent CLI（核心作品）

https://github.com/uncleTacitus/PaiCLI-Python

运行在终端中的 AI Agent CLI，面向真实项目开发场景。具备 ReAct、Plan-and-Execute、Multi-Agent 协作、MCP、RAG、记忆系统、快照恢复、Runtime API 等完整能力，覆盖 Agent 生产化的核心技术栈。

- **ReAct Agent 运行时**：完整实现 Observe → Think → Act 决策循环，支持 tool call、tool result、final output 和 usage 事件流式输出，基于 Rich + prompt-toolkit 的交互式终端渲染
- **Plan-and-Execute 模式**：独立 Planner 生成 DAG 任务图，按依赖关系分批并行执行；支持 HITL（人工确认）中断执行流程，确保关键操作安全可控
- **Multi-Agent 协作**：Planner（任务分解）、Worker（执行）、Reviewer（审查）三种 Agent 角色，支持依赖调度、并行 Worker 和 Review 重试机制
- **MCP 客户端**：支持 stdio 和 Streamable HTTP 两种传输方式的 MCP Server 接入；Agent 自身也可作为 MCP Server 暴露内置工具
- **RAG 与代码索引**：基于 Embedding + SQLite 的语义检索，支持代码分块（文件/类/方法粒度）、AST 解析、代码关系图谱（继承/实现/调用关系）
- **记忆系统**：短期记忆管理对话上下文；长期记忆通过 SQLite 持久化关键事实，跨会话复用；Agent Run 前后自动创建快照，支持恢复现场
- **Skill 系统**：支持内置/用户级/项目级 Skill，三级渐进式加载策略
- **Runtime API**：提供线程、turn（对话轮次）、事件日志和持久化后台任务能力；支持本地/远程图片输入及模型能力自动降级
- **技术栈**：Python 3.12 / asyncio / OpenAI-compatible API / Rich / prompt-toolkit / SQLite / MCP SDK

### PaiAgent — 企业级 AI Agent 工作流编排平台

https://github.com/uncleTacitus/PaiAgent

可视化拖拽编排的 AI 工作流平台，支持双引擎执行、多 LLM 提供商集成、Skill 技能系统，实现从 LLM 节点到工具调用的全链路 Agent 运行。

- **ReAct Agent 运行时**：实现完整的 Agent 决策循环，集成 Tool Registry 机制支持工具的注册、发现与动态调用；JSON 决策解析将 LLM 非结构化输出转为结构化 action，支持多步骤推理与 Token 用量追踪
- **双引擎架构**：DAG 引擎（Kahn 拓扑排序 + DFS 环检测）与 LangGraph4j 状态图引擎（Condition Branch + StateGraph + State Manager），通过 EngineSelector 工厂模式动态路由
- **Skill 技能系统**：基于 YAML Frontmatter 的 SKILL.md 声明机制，三段式加载策略（summary → detail → reference），支持在 Agent 节点中按需加载领域技能
- **多 LLM 提供商集成**：通过 Spring AI ChatClientFactory 动态创建客户端，统一接入 OpenAI、DeepSeek、Qwen、ZhiPu、GLM
- **可视化编排**：ReactFlow 构建的 Workflow 编辑器，拖拽式节点配置 + SSE 实时执行进度推送至前端 Debug Drawer
- **技术栈**：Spring Boot 3.4 / Java 21 / LangGraph4j / MyBatis-Plus / MySQL 8.0 / React 18 / TypeScript / ReactFlow / Redis

### PaiSmart — 企业级 AI 知识库与 RAG 检索系统

https://github.com/uncleTacitus/PaiSmart

基于检索增强生成（RAG）技术的企业级 AI 知识库管理系统，支持多租户、多格式文档上传与智能问答。

- **RAG 全流程实现**：文档解析（Apache Tika 多格式支持）→ Chunk → Embedding（支持本地 Ollama + 远程 API）→ Elasticsearch 索引 → 语义检索 → LLM 生成带引用回答
- **实时数据同步**：Kafka 消息队列异步处理文档索引，WebSocket 实时推送索引进度与问答结果
- **多租户架构**：Spring Security + JWT 鉴权体系，MinIO 对象存储，Redis 缓存加速
- **多模型兼容**：支持 DeepSeek API、本地 Ollama、豆包 Embedding 等多种 AI 集成方案
- **技术栈**：Spring Boot 3.4 / Java 17 / Elasticsearch / Kafka / Spring Data JPA / Vue 3 / TypeScript / MinIO / Docker

### 大宅门推客系统 v2 — 家装分销与线索运营平台（公司真实项目） 2025.06 - 至今

基于 FastAPI + SQLAlchemy 2.0 异步栈，为家装行业分销与线索运营平台提供 142+ 路由的后端服务，覆盖多角色、线索全生命周期、奖励佣金体系与微信支付。

- **多角色权限与数据隔离**：落地三层守卫（RequirePermission / RequireCertified / RequireIdentityVerified），替代 v1 硬编码权限；实现 CompanyScope 公司级数据隔离，总公司自动聚合分公司数据
- **线索状态机**：将线索状态从单值重构为 11 位状态标记 + 奖励关联位，奖励触发天然幂等，主管恢复不重复发奖
- **资金一致性**：设计多级佣金链路；提现 ≤400 元直接微信转账，>400 元主管审核；5 分钟周期主动对账提现与微信支付状态
- **v1 → v2 生产迁移**：编写 17 步 Alembic 迁移脚本，完成全量生产数据迁移并验证
- **AI 数据图表直查（亮点）**：为企微机器人设计 Data Chart Router 层，对中文查询做正则意图匹配，命中后直读 PostgreSQL 生成 SVG/PNG 图表返回，绕过 LLM；未命中走 OpenClaw Gateway SSE 流式回复，显著降低数据查询延迟与幻觉
- **技术栈**：Python 3.12 / FastAPI 0.115 / Pydantic v2 / SQLAlchemy 2.0 async / asyncpg / PostgreSQL 16 / Redis 7 / Alembic / 微信支付 / 腾讯云 COS & SMS

---

## 教育背景

**武汉学院** | 本科 | 网络工程专业 2019.09 - 2023.06

---

## 个人总结

- **Agent 工程能力**：主导开发 PaiCLI-Python（终端 Agent CLI）、PaiAgent（工作流编排平台）、PaiSmart（RAG 知识库）三个开源项目，完整覆盖 ReAct / Plan-and-Execute / Multi-Agent / MCP / RAG / Memory / Skill 等 Agent 生产化核心技术栈
- **Python/Java 双栈交付**：Python 端（FastAPI 异步栈 + Agent CLI 完整工程）与 Java 端（Spring Cloud 微服务 + LangGraph4j 编排引擎 + RAG 知识库）均具备独立交付公司级项目的能力
- **AI 与业务融合**：在公司项目中落地 AI 数据图表直查系统（规则引擎前置过滤 + LLM 兜底的生产级 AI 方案），深刻理解生产环境 AI 系统的延迟、幻觉、成本三大核心问题
- **技术与业务双视角**：2 年零售全链路运营经验，深谙从销售、供应链到库存管理的端到端业务流程，能将业务模型转化为 Agent 工作流设计
- **高性能与分布式**：多级缓存（Caffeine+Redis）QPS 3000+、RabbitMQ/RocketMQ 消息解耦、Seata AT 分布式事务、ELK 全链路可观测性
- **技术自驱**：持续投入数百小时系统学习 AI Agent 与分布式系统，具备极强的快速学习与独立交付能力
