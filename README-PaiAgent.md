<div align="center">

# PaiAgent

**企业级 AI 工作流可视化编排平台**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-21+-orange.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![LangGraph4j](https://img.shields.io/badge/LangGraph4j-1.8.0-purple.svg)](https://github.com/bsorrentino/langgraph4j)
[![React](https://img.shields.io/badge/React-18.x-61dafb.svg)](https://reactjs.org/)

</div>

> **PaiAgent** 是一个企业级的 AI 工作流可视化编排平台。通过拖拽式界面快速构建和编排 AI 工作流，支持 DAG 引擎和 LangGraph 状态图引擎双引擎架构，集成了多种大模型和 Skills 技能系统。

---

## 功能特性

- **双引擎驱动**：DAG 引擎（线性流水线） + LangGraph4j 状态图引擎（条件分支/循环/状态管理），按需切换
- **ReAct Agent 运行时**：完整的 Observe → Think → Act 循环，Tool Registry 机制
- **多 LLM 提供商**：统一接入 OpenAI、DeepSeek、Qwen、ZhiPu、GLM
- **Skills 技能系统**：YAML Frontmatter + Markdown 声明，三段式渐进加载
- **可视化编排**：ReactFlow 拖拽编辑器，SSE 实时执行反馈
- **JWT 鉴权**、MinIO 文件存储、执行记录持久化

---

## 架构

```
┌──────────────────────────────────────────────────┐
│                   Frontend                       │
│  ReactFlow Canvas  →  Node Panel  →  Debug       │
│  (拖拽编排)           (节点配置)     (实时输出)    │
└──────────────────────┬───────────────────────────┘
                       │ REST API / SSE
┌──────────────────────▼───────────────────────────┐
│              Backend (Spring Boot)               │
│                                                   │
│  ┌────────────┐    ┌──────────────────────────┐  │
│  │EngineSelector│──▶│   DAG Engine              │  │
│  │ (动态路由)   │   │   (Kahn 拓扑排序 +       │  │
│  │             │   │    DFS 环检测)             │  │
│  │             │   ├──────────────────────────┤  │
│  │             │──▶│   LangGraph4j 引擎        │  │
│  │             │   │   (StateGraph +           │  │
│  │             │   │    Condition Branch)       │  │
│  └────────────┘    └────────────┬─────────────┘  │
│                                 │                 │
│  ┌──────────────────────────────▼──────────────┐ │
│  │            Node Executors                   │ │
│  │  LLM  │  ReAct Agent  │  Input  │  Output  │ │
│  │  TTS  │  DeepSeek     │  Qwen   │  ZhiPu   │ │
│  └─────────────────────────────────────────────┘ │
│                                                   │
│  ┌────────────┐   ┌────────────┐  ┌───────────┐ │
│  │ Skills     │   │ JWT Auth   │  │ MinIO     │ │
│  │ 系统       │   │ 鉴权       │  │ 文件存储  │ │
│  └────────────┘   └────────────┘  └───────────┘ │
└──────────────────────────────────────────────────┘
```

---

## 快速开始

### 后端

```bash
cd backend
# 配置环境变量
cp .env.example .env
# 启动
./mvnw spring-boot:run
```

### 前端

```bash
cd frontend
npm install
npm run dev
```

### 数据库

```bash
mysql -u root -p < backend/src/main/resources/schema.sql
```

---

## 项目结构

```
PaiAgent/
├── backend/
│   └── src/main/java/com/paiagent/
│       ├── engine/
│       │   ├── dag/           # DAG 引擎（DAGParser + 拓扑排序）
│       │   ├── langgraph/     # LangGraph4j 引擎（StateGraph + GraphBuilder）
│       │   ├── executor/      # 节点执行器（LLM、ReAct、TTS、Input、Output）
│       │   ├── skill/         # Skill 系统（SKILL.md + 三段式加载）
│       │   ├── agent/         # 工具注册（Tool Registry）
│       │   └── llm/           # LLM 客户端工厂
│       ├── controller/        # REST API
│       ├── service/           # 业务逻辑
│       └── mapper/            # 数据访问
├── frontend/
│   └── src/
│       ├── components/        # ReactFlow 编辑器、节点面板、调试面板
│       ├── pages/             # 页面（Login、Main、Editor）
│       └── api/               # API 客户端
├── docs/                      # 文档
└── README.md
```

---

## 技术栈

### 后端
- Spring Boot 3.4, Java 21, MyBatis-Plus, MySQL 8.0
- LangGraph4j 1.8.0-beta3, Spring AI 1.0.0-M5
- FastJSON2, JJWT 0.12.7

### 前端
- React 18, TypeScript 5.6, Vite 6
- ReactFlow (@xyflow/react), Ant Design 6, Tailwind CSS 4
- Zustand 5

---

## 许可证

MIT
