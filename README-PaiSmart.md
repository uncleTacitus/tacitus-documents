<div align="center">

# PaiSmart — 派聪明

**企业级 AI 知识库与 RAG 检索系统**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-17+-green.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Elasticsearch](https://img.shields.io/badge/Elasticsearch-8.10-00bfb3.svg)](https://www.elastic.co/)
[![Vue](https://img.shields.io/badge/Vue-3.x-4fc08d.svg)](https://vuejs.org/)

</div>

> **PaiSmart（派聪明）** 是一个企业级的 AI 知识库管理系统，基于检索增强生成（RAG）技术。支持多格式文档上传、智能 chunk、向量化索引和语义检索问答。多租户架构，通过自然语言查询知识库，获得基于自身文档的 AI 生成响应。

---

## 功能特性

- **RAG 全流程**：文档上传 → 解析（Apache Tika）→ Chunk → Embedding → Elasticsearch 索引 → 语义检索 → LLM 生成带引用回答
- **多格式文档支持**：PDF、Word、Excel、Markdown、代码文件等
- **多 Embedding 方案**：支持本地 Ollama + 远程 API（豆包等）
- **多 LLM 支持**：DeepSeek API、本地 Ollama
- **实时同步**：Kafka 异步消息驱动文档索引，WebSocket 推送进度
- **多租户**：Spring Security + JWT 鉴权，数据隔离
- **可视化前端**：Vue 3 + TypeScript，拖拽上传 + 即时预览

---

## 架构

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (Vue 3)                   │
│   文档上传  →  知识库管理  →  智能问答   →  结果展示   │
└──────────────────────┬──────────────────────────────┘
                       │ REST API / WebSocket
┌──────────────────────▼──────────────────────────────┐
│              Backend (Spring Boot)                   │
│                                                      │
│  ┌─────────┐  ┌───────────┐  ┌───────────────────┐  │
│  │ 文档解析 │→│ Chunk +   │→│ Embedding + 索引    │  │
│  │ (Tika)  │  │ 分段策略  │  │ (Ollama/远程API)   │  │
│  └─────────┘  └───────────┘  └────────┬──────────┘  │
│                                       │              │
│  ┌────────────────────────────────────▼───────────┐  │
│  │           Semantic Search (ES) + Reranker      │  │
│  └────────────────────────────────────┬───────────┘  │
│                                       │              │
│  ┌────────────────────────────────────▼───────────┐  │
│  │        LLM Answer Generation                   │  │
│  │        (DeepSeek / Ollama / ...)               │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌─────────┐  ┌─────────┐  ┌────────┐  ┌─────────┐  │
│  │ Kafka   │  │ Redis   │  │ MinIO  │  │ JWT Auth│  │
│  │ 异步消息│  │ 缓存    │  │ 存储   │  │ 鉴权    │  │
│  └─────────┘  └─────────┘  └────────┘  └─────────┘  │
└──────────────────────────────────────────────────────┘
```

---

## 快速开始

### 后端

```bash
# 启动依赖服务（ES、MySQL、Redis、Kafka、MinIO）
docker-compose up -d

# 启动后端
cd backend
./mvnw spring-boot:run
```

### 前端

```bash
cd frontend
pnpm install
pnpm dev
```

---

## 技术栈

### 后端
- Spring Boot 3.4, Java 17, Spring Data JPA, MySQL 8.0
- Elasticsearch 8.10, Apache Kafka, Redis, MinIO
- Apache Tika（文档解析）, Spring Security + JWT
- DeepSeek API / Ollama, WebSocket

### 前端
- Vue 3, TypeScript, Vite
- Naive UI, Pinia, Vue Router
- UnoCSS, Iconify

---

## 许可证

MIT
