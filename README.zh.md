# The Mother of AI Project
## 第一阶段 RAG 系统：arXiv Paper Curator

> 本文为 [README.md](./README.md) 的一对一中文译本。

<div align="center">
  <h3>以学习者为中心的生产级 RAG 系统实践之旅</h3>
  <p>通过动手实现，从零开始学习构建现代 AI 系统</p>
  <p>掌握当前最受欢迎的 AI 工程技能：<strong>RAG（检索增强生成）</strong></p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12+-blue.svg" alt="Python Version">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-green.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenSearch-2.19-orange.svg" alt="OpenSearch">
  <img src="https://img.shields.io/badge/Docker-Compose-blue.svg" alt="Docker">
  <img src="https://img.shields.io/badge/Status-Week%207%20Advanced%20Features-brightgreen.svg" alt="Status">
</p>

</br>

<p align="center">
  <a href="#-关于本课程">
    <img src="static/mother_of_ai_project_rag_architecture.gif" alt="RAG 架构" width="700">
  </a>
</p>

## 📖 关于本课程

这是一个**以学习者为中心的项目**：你将构建一套完整的研究助手系统，自动抓取学术论文、理解其内容，并用先进的 RAG 技术回答你的研究问题。

**arXiv Paper Curator** 会教你用**行业最佳实践构建生产级 RAG 系统**。不同于一上来就做向量检索的教程，我们走**专业路径**：先掌握关键词检索基础，再用向量增强，形成混合检索。

> **🎯 专业路径的不同之处：** 我们按成功公司的方式构建 RAG——在扎实的搜索基础之上用 AI 增强，而不是忽略搜索基本功的 AI 优先路线。

课程结束时，你将拥有自己的 AI 研究助手，以及在任意领域构建生产级 RAG 系统的深度技术能力。

### **🎓 你将构建什么**

- **第 1 周：** 完整基础设施：Docker、FastAPI、PostgreSQL、OpenSearch 和 Airflow
- **第 2 周：** 自动从 arXiv 抓取并解析学术论文的数据管道
- **第 3 周：** 带过滤与相关性打分的生产级 BM25 关键词检索
- **第 4 周：** 智能切块 + 关键词与语义理解结合的混合检索
- **第 5 周：** 完整 RAG 流水线：本地 LLM、流式响应和 Gradio 界面
- **第 6 周：** 生产监控：Langfuse tracing 与 Redis 缓存以优化性能
- **第 7 周：** **基于 LangGraph 的 Agentic RAG，以及可移动访问的 Telegram Bot**

---

## 🏗️ 系统架构演进

### 第 7 周：Agentic RAG 与 Telegram Bot 集成
<div align="center">
  <img src="static/week7_telegram_and_agentic_ai.png" alt="第 7 周 Telegram 与 Agentic AI 架构" width="800">
  <p><em>完整第 7 周架构：Telegram bot 与 agentic RAG 系统的集成</em></p>
</div>

### LangGraph Agentic RAG 工作流
<div align="center">
  <img src="static/langgraph-mermaid.png" alt="LangGraph Agentic RAG 流程" width="800">
  <p><em>详细的 LangGraph 工作流：决策节点、文档打分与自适应检索</em></p>
</div>


**第 7 周代码 walkthrough + 博客：** [Agentic RAG with LangGraph and Telegram](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram) 

**第 7 周的关键创新：**
- **智能决策：** Agent 评估并调整检索策略
- **文档打分：** 用语义评估自动判断相关性
- **查询改写：** 结果不足时自适应优化查询
- **护栏：** 域外检测，防止幻觉
- **移动访问：** Telegram bot，任意设备上对话式 AI
- **可解释性：** 完整推理步骤追踪，便于调试与建立信任

---

## 🚀 快速开始

### **📋 前置条件**
- **Docker Desktop**（含 Docker Compose）  
- **Python 3.12+**
- **UV 包管理器**（[安装指南](https://docs.astral.sh/uv/getting-started/installation/)）
- **8GB+ RAM** 以及 **20GB+ 可用磁盘空间**

### **⚡ 开始使用**

```bash
# 1. Clone and setup
git clone <repository-url>
cd arxiv-paper-curator

# 2. Configure environment (IMPORTANT!)
cp .env.example .env
# The .env file contains all necessary configuration for OpenSearch, 
# arXiv API, and service connections. Defaults work out of the box.
# You need to add Jina embeddings free api key and langfuse keys (check the blogs)

# 3. Install dependencies
uv sync

# 4. Start all services
docker compose up --build -d

# 5. Verify everything works
curl http://localhost:8000/api/v1/health
```

### **📚 每周学习路径**

| 周次 | 主题 | 博客 | 代码发布 |
|------|-------|-----------|--------------|
| **第 0 周** | The Mother of AI project - 6 个阶段 | [The Mother of AI project](https://jamwithai.substack.com/p/the-mother-of-ai-project) | - |
| **第 1 周** | 基础设施奠基 | [The Infrastructure That Powers RAG Systems](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag) | [week1.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week1.0) |
| **第 2 周** | 数据摄取管道 | [Building Data Ingestion Pipelines for RAG](https://jamwithai.substack.com/p/bringing-your-rag-system-to-life) | [week2.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week2.0) |
| **第 3 周** | OpenSearch 入库与 BM25 检索 | [The Search Foundation Every RAG System Needs](https://jamwithai.substack.com/p/the-search-foundation-every-rag-system) | [week3.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week3.0) |
| **第 4 周** | **切块与混合检索** | [The Chunking Strategy That Makes Hybrid Search Work](https://jamwithai.substack.com/p/chunking-strategies-and-hybrid-rag) | [week4.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week4.0) |
| **第 5 周** | **完整 RAG 系统** | [The Complete RAG System](https://jamwithai.substack.com/p/the-complete-rag-system) | [week5.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week5.0) |
| **第 6 周** | **生产监控与缓存** | [Production-ready RAG: Monitoring & Caching](https://jamwithai.substack.com/p/production-ready-rag-monitoring-and) | [week6.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week6.0) |
| **第 7 周** | **Agentic RAG 与 Telegram Bot** | [Agentic RAG with LangGraph and Telegram](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram) | [week7.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week7.0) |

**📥 克隆某一周的发布版本：**
```bash
# Clone a specific week's code
git clone --branch <WEEK_TAG> https://github.com/jamwithai/arxiv-paper-curator
cd arxiv-paper-curator
uv sync
docker compose down -v
docker compose up --build -d

# Replace <WEEK_TAG> with: week1.0, week2.0, etc.
```

### **📊 访问各服务**

| 服务 | URL | 用途 |
|---------|-----|---------|
| **API 文档** | http://localhost:8000/docs | 交互式 API 测试 |
| **Gradio RAG 界面** | http://localhost:7861 | 友好的聊天界面 |
| **Langfuse Dashboard** | http://localhost:3000 | RAG 流水线监控与 tracing |
| **Airflow Dashboard** | http://localhost:8080 | 工作流管理 |
| **OpenSearch Dashboards** | http://localhost:5601 | 混合检索引擎 UI |

#### **注意**：Airflow 用户名和密码见 `airflow/simple_auth_manager_passwords.json.generated`
---

## 📚 第 1 周：基础设施奠基 ✅

**从这里开始！** 掌握支撑现代 RAG 系统的基础设施。

### **🎯 学习目标**
- 用 Docker Compose 完成完整基础设施搭建
- FastAPI 开发：自动文档与健康检查
- PostgreSQL 数据库配置与管理
- OpenSearch 混合检索引擎搭建
- Ollama 本地 LLM 服务配置
- 服务编排与健康监控
- 带代码质量工具的专业开发环境

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week1_infra_setup.png" alt="第 1 周基础设施搭建" width="800">
</p>

**基础设施组件：**
- **FastAPI**：带异步支持的 REST 接口（端口 8000）  
- **PostgreSQL 16**：论文元数据存储（端口 5432）
- **OpenSearch 2.19**：带 Dashboards 的检索引擎（端口 9200、5601）
- **Apache Airflow 3.0**：工作流编排（端口 8080）
- **Ollama**：本地 LLM 服务器（端口 11434）

### **📓 搭建指南**

```bash
# Launch the Week 1 notebook
uv run jupyter notebook notebooks/week1/week1_setup.ipynb
```

**完成指南：** 跟随 [第 1 周 notebook](notebooks/week1/week1_setup.ipynb) 进行动手搭建与验证。

### **📖 深入阅读**
**博客：** [The Infrastructure That Powers RAG Systems](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag) - 详细 walkthrough 与生产实践洞察

---

## 📚 第 2 周：数据摄取管道 ✅

**在第 1 周基础设施之上：** 学习自动抓取、处理并存储学术论文。

### **🎯 学习目标**
- 带限流与重试逻辑的 arXiv API 集成
- 使用 Docling 解析科学 PDF
- 用 Apache Airflow 做自动化数据摄取管道
- 元数据提取与存储工作流
- 从 API 到数据库的完整论文处理

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week2_data_ingestion_flow.png" alt="第 2 周数据摄取架构" width="800">
</p>

**数据管道组件：**
- **MetadataFetcher**：🎯 协调整条管道的主编排器
- **ArxivClient**：带限流与重试的论文抓取
- **PDFParserService**：基于 Docling 的科学文档处理  
- **Airflow DAGs**：每日自动论文摄取工作流
- **PostgreSQL Storage**：结构化的论文元数据与正文

### **📓 实现指南**

```bash
# Launch the Week 2 notebook  
uv run jupyter notebook notebooks/week2/week2_arxiv_integration.ipynb
```

**完成指南：** 跟随 [第 2 周 notebook](notebooks/week2/week2_arxiv_integration.ipynb) 进行动手实现与验证。

### **📖 深入阅读**
**博客：** [Building Data Ingestion Pipelines for RAG](https://jamwithai.substack.com/p/bringing-your-rag-system-to-life) - arXiv API 集成与 PDF 处理

---

## 📚 第 3 周：先做关键词检索——关键基础

**在第 1–2 周基础之上：** 实现专业 RAG 系统所依赖的关键词检索基础。

### **🎯 学习目标**
- 为什么关键词检索对 RAG 必不可少（基础优先）
- OpenSearch 索引管理、mapping 与检索优化
- BM25 算法以及有效关键词检索背后的数学
- 用 Query DSL 构建带过滤与 boosting 的复杂查询
- 用搜索分析衡量相关性与性能
- 真实公司使用的生产模式

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week3_opensearch_flow.png" alt="第 3 周 OpenSearch 流程架构" width="800">
</p>

**检索基础设施组件：**
- **OpenSearch Service**：`src/services/opensearch/` - 专业检索服务实现
- **Search API**：`src/routers/search.py` - 带 BM25 打分的检索 API
- **学习材料**：`notebooks/week3/` - 完整 OpenSearch 集成指南
- **质量指标**：精确率、召回率与相关性打分

### **📓 搭建指南**

```bash
# Launch the Week 3 notebook
uv run jupyter notebook notebooks/week3/week3_opensearch.ipynb
```

**完成指南：** 跟随 [第 3 周 notebook](notebooks/week3/week3_opensearch.ipynb) 动手搭建 OpenSearch 并实现 BM25 检索。

### **📖 深入阅读**
**博客：** [The Search Foundation Every RAG System Needs](https://jamwithai.substack.com/p/the-search-foundation-every-rag-system) - 用 OpenSearch 完整实现 BM25

---

## 📚 第 4 周：切块与混合检索——语义层

**在第 3 周基础之上：** 加入让检索真正智能的语义层。

### **🎯 学习目标**
- 基于章节的智能文档切块
- 生产级 embedding：Jina AI 集成与回退策略
- 掌握混合检索：用 RRF 融合关键词 + 语义检索
- 统一 API 设计：单一端点支持多种检索模式
- 不同检索方式的性能分析与权衡

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week4_hybrid_opensearch.png" alt="第 4 周混合检索架构" width="800">
</p>

**混合检索基础设施组件：**
- **Text Chunker**：`src/services/indexing/text_chunker.py` - 感知章节结构、带 overlap 的切块
- **Embeddings Service**：`src/services/embeddings/` - 基于 Jina AI 的生产级 embedding 管道
- **Hybrid Search API**：`src/routers/hybrid_search.py` - 支持全部模式的统一检索 API
- **学习材料**：`notebooks/week4/` - 完整混合检索实现指南

### **📓 搭建指南**

```bash
# Launch the Week 4 notebook
uv run jupyter notebook notebooks/week4/week4_hybrid_search.ipynb
```

**完成指南：** 跟随 [第 4 周 notebook](notebooks/week4/week4_hybrid_search.ipynb) 进行动手实现与验证。

### **📖 深入阅读**
**博客：** [The Chunking Strategy That Makes Hybrid Search Work](https://jamwithai.substack.com/p/chunking-strategies-and-hybrid-rag) - 生产级切块与 RRF 融合实现

---

## 📚 第 5 周：接入 LLM 的完整 RAG 流水线

**在第 4 周混合检索之上：** 加入把检索变成智能对话的 LLM 层。

### **🎯 学习目标**
- 用 Ollama 集成本地 LLM，数据完全私有
- 性能优化：prompt 缩短 80%（速度提升约 6 倍）
- 用 Server-Sent Events 实现流式实时响应
- 双 API 设计：标准端点与流式端点
- 带高级参数控制的交互式 Gradio 界面

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week5_complete_rag.png" alt="第 5 周完整 RAG 系统架构" width="900">
</p>

**完整 RAG 基础设施组件：**
- **RAG Endpoints**：`src/routers/ask.py` - 双端点（`/api/v1/ask` + `/api/v1/stream`）
- **Ollama Service**：`src/services/ollama/` - 带优化 prompt 的 LLM 客户端
- **System Prompt**：`src/services/ollama/prompts/rag_system.txt` - 针对学术论文优化
- **Gradio Interface**：`src/gradio_app.py` - 支持流式的交互式 Web UI
- **Launcher Script**：`gradio_launcher.py` - 一键启动脚本（端口 7861）

### **📓 搭建指南**

```bash
# Launch the Week 5 notebook
uv run jupyter notebook notebooks/week5/week5_complete_rag_system.ipynb

# Launch Gradio interface
uv run python gradio_launcher.py
# Open http://localhost:7861
```

**完成指南：** 跟随 [第 5 周 notebook](notebooks/week5/week5_complete_rag_system.ipynb) 动手接入 LLM 并实现 RAG 流水线。

### **📖 深入阅读**
**博客：** [The Complete RAG System](https://jamwithai.substack.com/p/the-complete-rag-system) - 完整 RAG 系统：本地 LLM 集成与优化技巧

---

## 📚 第 6 周：生产监控与缓存

**在第 5 周完整 RAG 系统之上：** 加入可观测性、性能优化与生产级监控。

### **🎯 学习目标**
- 集成 Langfuse，对整条 RAG 流水线做 tracing
- Redis 缓存策略：智能 cache key 与 TTL 管理
- 用实时 dashboard 监控延迟与成本
- 可观测性与优化的生产模式
- 成本分析与 LLM 用量优化（缓存带来 150–400 倍加速）

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week6_monitoring_and_caching.png" alt="第 6 周监控与缓存架构" width="900">
</p>

**生产基础设施组件：**
- **Langfuse Service**：`src/services/langfuse/` - 完整 tracing 集成，含 RAG 专用指标
- **Cache Service**：`src/services/cache/` - Redis 客户端：精确匹配缓存与优雅回退
- **Updated Endpoints**：`src/routers/ask.py` - 已接入 tracing 与缓存中间件
- **Docker Config**：`docker-compose.yml` - 新增 Redis 服务与本地 Langfuse 实例
- **学习材料**：`notebooks/week6/` - 完整监控与缓存实现指南

### **📓 搭建指南**

```bash
# Launch the Week 6 notebook
uv run jupyter notebook notebooks/week6/week6_cache_testing.ipynb
```

**完成指南：** 跟随 [第 6 周 notebook](notebooks/week6/week6_cache_testing.ipynb) 动手实现 Langfuse tracing 与 Redis 缓存。

### **📖 深入阅读**
**博客：** [Production-ready RAG: Monitoring & Caching](https://jamwithai.substack.com/p/production-ready-rag-monitoring-and) - 带监控与缓存的生产级 RAG

---

## 📚 第 7 周：LangGraph Agentic RAG 与 Telegram Bot

**在第 6 周生产系统之上：** 加入智能推理、多步决策，以及面向移动优先的 Telegram bot 交互。

### **🎯 学习目标**
- 用 LangGraph 做带决策节点的状态机式 agent 编排
- 实现护栏：查询校验与领域边界检测
- 用语义相关性评估做文档打分
- 查询改写：自动优化查询以提升检索
- 自适应检索：多次尝试与智能回退
- Telegram bot 集成：异步操作与错误处理
- 推理透明：暴露 agent 的决策过程

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week7_telegram_and_agentic_ai.png" alt="第 7 周 Agentic RAG 与 Telegram 架构" width="900">
</p>

**Agentic RAG 基础设施组件：**
- **Agent Nodes**：`src/services/agents/nodes/` - 护栏、检索、打分、改写与生成节点
- **Workflow Orchestration**：`src/services/agents/agentic_rag.py` - LangGraph 工作流协调
- **Telegram Bot**：`src/services/telegram/` - 命令处理与消息处理
- **Agentic Endpoint**：`src/routers/agentic_ask.py` - Agentic RAG API
- **学习材料**：`notebooks/week7/` - 第 7 周学习材料与示例

### **📓 搭建指南**

```bash
# Launch the Week 7 notebook
uv run jupyter notebook notebooks/week7/week7_agentic_rag.ipynb
```

**完成指南：** 跟随 [第 7 周 notebook](notebooks/week7/week7_agentic_rag.ipynb) 动手实现 LangGraph agentic RAG 与 Telegram bot。

### **📖 深入阅读**
**博客：** [Agentic RAG with LangGraph and Telegram](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram) - 构建具备决策、自适应检索与移动访问能力的智能 agent

---

## ⚙️ 配置

**设置：**
```bash
cp .env.example .env
# Edit .env for your environment
```

**关键变量：**
- `JINA_API_KEY` - 第 4 周及以后需要（带 embedding 的混合检索）
- `TELEGRAM__BOT_TOKEN` - 第 7 周需要（Telegram bot 集成）
- `LANGFUSE__PUBLIC_KEY` & `LANGFUSE__SECRET_KEY` - 第 6 周可选（监控）

**完整配置：** 所有可用选项与详细说明见 [.env.example](.env.example)。

---

## 🔧 参考与开发指南

### **🛠️ 技术栈**

| 服务 | 用途 | 状态 |
|---------|---------|--------|
| **FastAPI** | 带自动文档的 REST API | ✅ 就绪 |
| **PostgreSQL 16** | 论文元数据与正文存储 | ✅ 就绪 |
| **OpenSearch 2.19** | 混合检索引擎（BM25 + 向量） | ✅ 就绪 |
| **Apache Airflow 3.0** | 工作流自动化 | ✅ 就绪 |
| **Jina AI** | Embedding 生成（第 4 周） | ✅ 就绪 |
| **Ollama** | 本地 LLM 服务（第 5 周） | ✅ 就绪 |
| **Redis** | 高性能缓存（第 6 周） | ✅ 就绪 |
| **Langfuse** | RAG 流水线可观测性（第 6 周） | ✅ 就绪 |

**开发工具：** UV、Ruff、MyPy、Pytest、Docker Compose

### **🏗️ 项目结构**

```
arxiv-paper-curator/
├── src/                    # Main application code
│   ├── routers/            # API endpoints (search, ask, papers)
│   ├── services/           # Business logic (opensearch, ollama, agents, cache)
│   ├── models/             # Database models (SQLAlchemy)
│   ├── schemas/            # Pydantic validation schemas
│   └── config.py           # Environment configuration
├── notebooks/              # Weekly learning materials (week1-7)
├── airflow/                # Workflow orchestration (DAGs)
├── tests/                  # Test suite
└── compose.yml             # Docker service orchestration
```

### **📡 API 端点参考**

| 端点 | 方法 | 说明 | 周次 |
|----------|--------|-------------|------|
| `/health` | GET | 服务健康检查 | 第 1 周 |
| `/api/v1/papers` | GET | 列出已存储论文 | 第 2 周 |
| `/api/v1/papers/{id}` | GET | 获取指定论文 | 第 2 周 |
| `/api/v1/search` | POST | BM25 关键词检索 | 第 3 周 |
| `/api/v1/hybrid-search/` | POST | 混合检索（BM25 + 向量） | **第 4 周** |

**API 文档：** 打开 http://localhost:8000/docs 使用交互式 API 浏览器

### **🔧 常用命令**

#### **使用 Makefile**（推荐）
```bash
# View all available commands
make help

# Quick workflow
make start         # Start all services
make health        # Check all services health
make test          # Run tests
make stop          # Stop services
```

#### **全部可用命令**
| 命令 | 说明 |
|---------|-------------|
| `make start` | 启动全部服务 |
| `make stop` | 停止全部服务 |
| `make restart` | 重启全部服务 |
| `make status` | 查看服务状态 |
| `make logs` | 查看服务日志 |
| `make health` | 检查全部服务健康状态 |
| `make setup` | 安装 Python 依赖 |
| `make format` | 格式化代码 |
| `make lint` | Lint 与类型检查 |
| `make test` | 运行测试 |
| `make test-cov` | 带覆盖率运行测试 |
| `make clean` | 清理一切 |

#### **直接命令**（备选）
```bash
# If you prefer using commands directly
docker compose up --build -d    # Start services
docker compose ps               # Check status
docker compose logs            # View logs
uv run pytest                 # Run tests
```

### **🎓 目标读者**
| 对象 | 原因 |
|-----|-----|
| **AI/ML 工程师** | 学习超越教程的生产级 RAG 架构 |
| **软件工程师** | 用最佳实践构建端到端 AI 应用 |
| **数据科学家** | 用现代工具实现生产级 AI 系统 |

---

## 🛠️ 故障排除

**常见问题：**
- **服务起不来？** 等 2–3 分钟，查看 `docker compose logs`
- **端口冲突？** 停掉占用 8000、8080、5432、9200 的其他服务
- **内存问题？** 提高 Docker Desktop 的内存配额

**获取帮助：**
- 查看第 1 周 notebook 中完整的故障排除章节
- 查看服务日志：`docker compose logs [service-name]`
- 完全重置：`docker compose down --volumes && docker compose up --build -d`

---

## 💰 成本结构

**本课程完全免费！** 可选服务只需极少费用：
- **本地开发：** $0（全部在本地运行）
- **可选云 API：** 外部 LLM 服务约 $2–5（若选择使用）

---

<div align="center">
  <h3>🎉 准备好开始你的 AI 工程之旅了吗？</h3>
  <p><strong>从第 1 周搭建 notebook 开始，构建你的第一个生产级 RAG 系统！</strong></p>
  
  <p><em>面向想掌握现代 AI 工程的学习者</em></p>
  <p><strong>由 <a href="https://www.linkedin.com/in/shirin-khosravi-jam/">Shirin Khosravi Jam</a> 与 <a href="https://www.linkedin.com/in/shantanuladhwe/">Shantanu Ladhwe</a> 用心打造</strong></p>
</div>

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=jamwithai/production-agentic-rag-course&type=Date)](https://star-history.com/#jamwithai/production-agentic-rag-course&Date)

---

## 📄 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件。
