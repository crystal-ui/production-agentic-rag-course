# Airflow 配置

> 本文为 [README.md](./README.md) 的一对一中文译本。

本目录包含 arXiv Paper Curator 项目的 Apache Airflow 配置与 DAG。

<p align="center">
  <img src="../static/week2_data_ingestion_flow.png" alt="第 2 周数据摄取架构" width="800">
</p>

## 当前设置（第 2 周）

### 生产就绪 DAG
- **hello_world_dag.py**：第 1 周基础健康检查 DAG
- **arxiv_paper_ingestion.py**：自动抓取与处理 arXiv 论文的主生产 DAG

### 生产管道特性
- **每日 arXiv 摄取**：自动抓取 CS.AI 论文
- **PDF 处理**：用 Docling 下载并解析论文
- **数据库存储**：在 PostgreSQL 中存储完整论文元数据与正文
- **错误处理**：全面的重试逻辑与错误报告
- **跨平台兼容**：适用于 macOS、Linux、WSL 和 Ubuntu

## 目录结构

```
airflow/
├── README.md                           # This file
├── Dockerfile                          # Custom Airflow container with dependencies
├── requirements-airflow.txt            # Python dependencies for DAGs
└── dags/
    ├── hello_world_dag.py             # Week 1 health check DAG
    ├── arxiv_paper_ingestion.py       # Week 2 production ingestion DAG
    └── arxiv_ingestion/
        └── tasks.py                   # Production pipeline tasks with async processing
```

## Docker 配置

### 跨平台兼容
Airflow 容器已配置为可跨平台部署：
- **用户配置**：以 `airflow` 用户（50000:0）运行，避免权限问题
- **Volume 管理**：日志使用 named volume，避免 bind mount 冲突
- **数据库集成**：连接到共享 PostgreSQL 实例
- **服务依赖**：自动初始化与健康检查

### 容器特性
- **Python 3.12** 搭配 Apache Airflow 2.10.3
- 通过 psycopg2 支持 **PostgreSQL**
- 用 Docling、Tesseract OCR 和 Poppler 工具做 **PDF 处理**
- 为遵守 arXiv API 而做的 **限流与重试逻辑**
- **异步处理**，以并发下载与解析获得最佳性能

## 使用方式

### Web 界面
- **URL**：http://localhost:8080
- **凭据**：容器初始化时自动生成
- **功能**：DAG 监控、任务日志、管道统计

### 生产 DAG（`arxiv_paper_ingestion`）
1. **环境准备**：验证服务并初始化缓存
2. **每日论文抓取**：获取前一天的论文（默认 10 篇）
3. **PDF 处理**：用 Docling 下载并解析 PDF
4. **失败 PDF 重试**：处理任何处理失败
5. **数据库存储**：存储带解析内容的完整论文数据
6. **OpenSearch 占位**：为第 3 周及以后的检索索引做准备
7. **每日报告**：生成全面的处理统计

### 管道性能
- **并发处理**：5 路并行下载、1 路解析（针对笔记本优化）
- **限流**：遵守 arXiv API 指南（3 秒间隔）
- **缓存**：PDF 本地缓存，避免重复下载
- **错误韧性**：个别论文失败时仍继续处理

## 配置

### 环境变量
```bash
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
AIRFLOW__CORE__EXECUTOR=LocalExecutor
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
PYTHONPATH=/opt/airflow/src
```

### 服务依赖
- **PostgreSQL**：论文元数据与正文存储
- **源代码**：从 `../src` 挂载以便访问服务
- **共享网络**：与 API 和数据库服务通信

## 第 2 周实现状态

### ✅ 已完成特性
- 带全部依赖的自定义 Docker 容器
- 带全面错误处理的生产级 arXiv 摄取 DAG
- 带并发控制的异步 PDF 处理管道
- 带完整内容存储的 PostgreSQL 集成
- 跨平台兼容（macOS、Linux、WSL、Ubuntu）
- 为遵守 arXiv API 的限流与重试逻辑
- 贯穿管道的详细日志与监控

### 🔄 第 3 周及以后路线图
- **OpenSearch 集成**：真实检索索引（当前为占位）
- **高级调度**：多种采集策略
- **监控与告警**：生产可观测性
- **规模优化**：面向生产负载的更高并发
