# 客服智能知识平台

基于 LLM 的企业级客服/售后支持智能知识平台，专注于产品手册检索、故障诊断引导、工单知识复用和智能话术推荐。

## 项目概述

本项目是一个面向客服/售后支持场景的 RAG 系统，从通用文档检索助手演进为专业的客服知识平台，解决四大核心痛点：

1. **知识检索效率低** — 客服人员无法快速找到产品手册中的答案
2. **故障排查无标准流程** — 缺乏结构化的故障诊断引导
3. **工单知识未复用** — 历史工单的解决方案散落各处
4. **新人上手慢** — 缺乏智能辅助和话术推荐

## 核心功能

### 📦 产品知识库管理
- **产品管理**：创建、编辑、删除产品，支持分类和版本管理
- **手册上传**：支持 PDF、DOCX、TXT 文件上传
- **异步处理**：RQ 任务队列 + Redis，后台完成文档解析、切片、向量生成
- **智能分块**：基于语义的文本切片策略
- **产品参数**：结构化存储产品规格参数，支持快速查询
- **故障 SOP**：录入故障排查标准操作流程

### 🎫 工单知识库
- **工单管理**：创建、查询、更新工单，支持状态流转（open→in_progress→resolved→closed）
- **相似工单匹配**：基于向量检索 + Reranker，自动推荐历史解决方案
- **知识复用**：历史工单解决方案自动沉淀到知识库

### 🔧 故障诊断引擎
- **症状匹配**：基于关键词匹配故障 SOP
- **引导式诊断**：多轮追问，按步骤引导排查
- **诊断记录**：保存诊断上下文，支持断点恢复
- **智能降级**：SOP 匹配失败时自动降级为通用知识检索

### 💬 智能客服对话
- **意图识别**：自动识别知识查询、故障诊断、工单查询三种意图
- **SSE 流式输出**：实时流式返回答问内容
- **引用卡片**：文本引用展示，标明知识来源
- **话术推荐**：基于检索结果生成专业客服话术
- **会话管理**：多轮对话、会话关联产品/工单

### 🛠️ 客服专用工具
- `product_spec_lookup`：查询产品参数/规格
- `ticket_search`：搜索相似历史工单
- `fault_diagnosis`：启动故障诊断流程
- `sop_lookup`：查询故障排查 SOP

## 技术架构

```
┌─────────────────────────────────────────────────────────┐
│                      Vue 3 单页应用                       │
│            Element Plus + Pinia + TypeScript            │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTP / SSE
┌─────────────────────▼───────────────────────────────────┐
│                    FastAPI REST API                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │  产品路由   │  │  知识路由   │  │  工单路由   │      │
│  │  对话路由   │  │  诊断路由   │  │  系统路由   │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│  ┌─────────────────────────────────────────────────┐    │
│  │                    服务层                          │    │
│  │  ProductService │ TicketService │ DiagnosisService │    │
│  │  CustomerServiceQA │ ChatService │ RetrievalService │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │                    仓储层                          │    │
│  │  ProductRepo │ TicketRepo │ FaultSOPRepo │ ChunkRepo │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                    基础设施层                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  向量存储 │  │   LLM    │  │   队列   │              │
│  │(pgvector)│  │(DashScope)│  │ (Redis) │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────────────────────────────────┘
```

## 技术栈

| 层级 | 技术选型 |
|------|----------|
| **前端** | Vue 3 + TypeScript + Vite + Element Plus + Pinia |
| **后端** | FastAPI + Pydantic + SQLAlchemy + Alembic + Uvicorn |
| **任务队列** | RQ (Redis Queue) + Redis |
| **数据库** | PostgreSQL + pgvector（向量检索）|
| **模型服务** | 通义千问（Qwen）+ DashScope Embedding + BGE Reranker |
| **文档解析** | PyMuPDF、pdfplumber、python-docx |

## 数据模型

### 核心业务模型

| 模型 | 说明 |
|------|------|
| Product | 产品（name, category, version, status） |
| ProductManual | 产品手册（关联 Product，替代原 Document） |
| ProductSpec | 产品参数/规格（结构化查询） |
| FaultSOP | 故障排查 SOP（含 symptoms/steps JSON） |
| Ticket | 工单（含状态流转：open→in_progress→resolved→closed） |

### 基础设施模型

| 模型 | 说明 |
|------|------|
| Chunk | 文档切片（新增 product_id, source_category） |
| Session | 会话（新增 product_id, ticket_id 关联） |
| Message | 消息（新增 diagnosis_context 诊断上下文） |
| Task | 异步任务 |

## 目录结构

```
├── frontend/                    # Vue 3 单页应用
│   ├── src/
│   │   ├── components/         # Vue 组件
│   │   │   ├── products/      # 产品管理组件
│   │   │   ├── knowledge/     # 知识库管理组件
│   │   │   ├── tickets/       # 工单管理组件
│   │   │   ├── chat/          # 客服对话组件
│   │   │   └── common/        # 通用组件
│   │   ├── services/           # API 调用层
│   │   ├── stores/            # Pinia 状态管理
│   │   └── types/             # TypeScript 类型定义
│   └── tests/                 # Vitest 单元测试
│
├── backend/                    # FastAPI 后端
│   ├── api/
│   │   ├── routes/            # API 路由（products, knowledge, tickets, chat, diagnosis, system）
│   │   └── schemas/           # Pydantic 模型
│   ├── app/
│   │   ├── models/           # SQLAlchemy 模型（Product, ProductManual, ProductSpec, FaultSOP, Ticket）
│   │   ├── repositories/      # 数据访问层
│   │   ├── services/          # 业务逻辑层（客服专用服务）
│   │   ├── orchestrators/     # 流程编排
│   │   ├── tasks/            # RQ 异步任务
│   │   └── tools/            # 客服专用工具
│   ├── infrastructure/        # 外部集成
│   │   ├── database/          # PostgreSQL 连接
│   │   ├── vector/           # pgvector 向量存储
│   │   ├── llm/              # LLM 模型调用
│   │   └── queue/            # Redis 队列
│   └── tests/                # 自动化测试
│
├── worker/                     # RQ 异步任务处理器
├── scripts/                    # 开发、测试、部署脚本
├── docs/                       # 设计文档和实施计划
└── memory-bank/               # 项目架构和进展记录
```

## 快速开始

### 环境要求

- Python 3.10+
- Node.js 18+
- Docker Desktop

### 启动步骤

```bash
# 1. 配置环境变量
copy .env.example .env
# 编辑 .env 填入 DASHSCOPE_API_KEY

# 2. 启动依赖容器
docker-compose up -d

# 3. 数据库迁移
python -m alembic upgrade head

# 4. 启动服务
start.bat dev
```

访问 http://127.0.0.1:5173

### 服务地址

| 服务 | 地址 |
|------|------|
| 前端应用 | http://127.0.0.1:5173 |
| 后端 API | http://127.0.0.1:8000 |
| API 文档 | http://127.0.0.1:8000/docs |

## 环境变量

| 配置项 | 说明 |
|--------|------|
| `DASHSCOPE_API_KEY` | 阿里云 DashScope API Key（必填） |
| `DATABASE_URL` | PostgreSQL 连接字符串 |
| `REDIS_URL` | Redis 连接字符串 |
| `FILE_STORAGE_PATH` | 文件存储路径 |
| `APP_ENV` | 应用环境（development/test/production） |

完整配置参考 `.env.example`。

## 测试

```bash
# 后端测试
python -m pytest backend/tests/ -v

# 前端测试
cd frontend && npm run test:unit

# 类型检查
cd frontend && npm run typecheck

# 代码检查
cd frontend && npm run lint
```

## 开发脚本

```bash
# 查看帮助
python run.py help

# 健康检查
python run.py health

# 运行烟测
python run.py smoke

# 部署验收
python run.py acceptance
```

## 项目亮点

- **客服场景定制**：从通用 RAG 演进为专业客服知识平台
- **前后端分离**：Vue 3 SPA + FastAPI REST API，SSE 流式通信
- **异步任务处理**：RQ + Redis 实现文档处理与模型调用的异步化
- **向量检索**：PostgreSQL + pgvector 支持高效语义检索
- **故障诊断引擎**：基于 SOP 的引导式故障排查
- **工单知识复用**：相似工单匹配，历史解决方案自动推荐
- **智能话术推荐**：基于检索结果生成专业客服话术
- **客服专用工具**：product_spec_lookup、ticket_search、fault_diagnosis、sop_lookup

## 演进历程

本项目经历了多个阶段的演进：

- **第一阶段**：搭建 FastAPI + Vue 3 + PostgreSQL + pgvector + RQ + Redis 基础架构
- **第二阶段**：实现 Tool Calling 机制（web_search、document_lookup）
- **第三阶段**：实现多模态 RAG（PDF 视觉元素提取、Qwen-VL 描述生成）
- **第四阶段**：实现 GraphRAG（Neo4j 知识图谱、图检索与向量检索双路召回）
- **第五阶段**：产品化最小闭环（健康检查、就绪检查、配置校验、部署验收）
- **第六阶段**：前端界面优化与用户体验提升
- **第七阶段**：文档与会话管理增强（标签系统、搜索、导出、置顶）
- **第八阶段**：客服场景重构（产品管理、工单知识库、故障诊断、话术推荐）

## 许可证

MIT
