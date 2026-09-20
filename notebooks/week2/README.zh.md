# 第 2 周：arXiv API 集成与 PDF 处理

> 本文为 [README.md](./README.md) 的一对一中文译本。

本目录包含 arXiv Paper Curator 项目第 2 周材料，重点是构建为核心数据摄取管道，把新鲜学术内容送入 RAG 系统。

## 内容

### `week2_arxiv_integration.ipynb`
一份完整的 Jupyter notebook，引导学生完成：

1. **基础设施校验**
   - 确认第 1 周全部服务运行正常
   - 容器健康检查与全新构建验证
   - 第 2 周组件的环境准备

2. **arXiv API 集成**
   - 构建带限流与重试逻辑的稳健客户端
   - 实现按日期过滤以定向检索论文
   - 按正确的 API 礼仪测试 CS.AI 类别搜索

<p align="center">
  <img src="../../static/week2_data_ingestion_flow.png" alt="第 2 周数据摄取架构" width="800">
</p>

**数据管道概览：**
- **MetadataFetcher**：🎯 协调整条管道的主编排器
- **ArxivClient**：带重试逻辑的限流抓取（3 秒间隔）
- **PDFParserService**：科学 PDF 解析与结构化内容抽取
- **PaperRepository**：带 upsert 操作的 PostgreSQL 集成
- **Airflow DAGs**：每日自动摄取工作流

3. **PDF 处理管道**
   - 下载并缓存 PDF，配合恰当的错误处理
   - 用 Docling 解析科学 PDF，抽取结构化内容
   - 解析失败时优雅回退

4. **数据库集成**
   - 把论文元数据与正文存入 PostgreSQL
   - 实现 upsert 逻辑以避免重复
   - 测试检索与查询操作

5. **完整管道测试**
   - 从 arXiv API 到数据库存储的端到端处理
   - 错误处理与优雅降级测试
   - 性能指标与成功率分析

6. **生产就绪**
   - Airflow DAG 状态验证
   - 错误日志与监控能力
   - 为每日自动摄取做好准备

## 学习目标

完成本周材料后，学员将能够：

- 掌握带正确限流与错误处理的 API 集成
- 学习科学文档的 PDF 处理技术
- 理解研究数据存储的数据库设计模式
- 构建带全面错误处理的稳健数据管道
- 获得异步 Python 编程模式经验
- 学习 Apache Airflow 的工作流自动化概念
- 培养生产级数据处理系统的技能

## 关键技术与服务

### 核心服务（本周构建）
- **arXiv API Client** - 以智能限流抓取 CS.AI 论文
- **PDF Parser (Docling)** - 从科学 PDF 抽取结构化内容
- **Metadata Fetcher** - 编排完整处理管道
- **Database Repository** - 用 SQLAlchemy 处理 PostgreSQL 操作

### 基础设施依赖（来自第 1 周）
- **PostgreSQL 16** - 论文元数据与正文存储
- **FastAPI** - 用于论文检索的 REST API
- **Apache Airflow** - 工作流编排与调度
- **Docker Compose** - 服务编排与组网

## 管道架构

```
arXiv Search Query → Rate Limited API Calls → PDF Downloads → Docling Parsing → Database Storage
        ↓                    ↓                    ↓              ↓               ↓
   Date Filtering    →  Retry Logic       →   Caching      → Structure    →  Upsert Logic
   Category Filter   →  Error Handling    →   Validation   → Extraction   →  Transactions
   Result Limiting   →  3s Rate Limit     →   Size Checks  → Metadata     →  Relationships
```

## 性能特征

**第 2 周系统能力：**
- **arXiv API**：约 20 篇/分钟（遵守 3 秒限流）
- **PDF 处理**：每篇 2–5 秒（取决于 PDF 复杂度）
- **数据库存储**：约 100 篇/秒（批量操作）
- **错误处理**：个别失败时仍优雅继续
- **成功率**：论文抓取 95%+，PDF 解析 80–90%

## 目标读者

本材料面向：
- 学习构建研究数据管道的**数据工程师**
- 对学术研究自动化感兴趣的**学生**
- 构建内容聚合系统的**开发者**
- 想自动化文献发现的**研究者**
- 任何构建生产级数据摄取管道的人

## 时间投入

- **全新容器构建**：10–15 分钟（第 2 周依赖所必需）
- **完成 notebook**：45–60 分钟
- **管道测试**：30–45 分钟
- **合计**：1.5–2 小时

## 📖 额外资源

**第 2 周博客：** [Building Robust Data Pipelines for Academic Research](https://jamwithai.substack.com/p/building-data-pipelines-academic-research)
- 深入 arXiv API 最佳实践
- 科学文档的 PDF 处理策略
- 生产系统的错误处理模式
- 研究元数据的数据库设计

## 重要说明

### 必须全新构建容器
第 2 周需要用新依赖重建容器：

```bash
# Shutdown and rebuild (REQUIRED for Week 2)
docker compose down
docker compose up --build

# This ensures:
# - New Python dependencies (docling, arxiv client)
# - Updated Airflow DAGs
# - Fresh service configurations
```

### PDF 处理预期
- 并非所有 PDF 都能解析成功（预期行为）
- Docling 对标准学术论文格式效果最好
- 系统会优雅处理失败并继续处理
- 学术 PDF 成功率 80–90% 是正常的

## 支持资源

如果遇到问题：
1. 确认已全新构建容器（`docker compose up --build`）
2. 查看 notebook 中的故障排除章节
3. 检查服务健康检查与日志
4. 确认第 1 周基础设施全部正常
5. 在 Jam With AI substack 聊天频道提问

## 下一步

完成第 2 周后，你将能够：
- 理解生产级数据管道架构
- 应对真实世界的 API 集成挑战
- 实现稳健的错误处理与监控
- 进入第 3 周：OpenSearch 集成与全文检索
- 更有信心地处理复杂数据处理工作流

## 成功标准

✅ **第 2 周完成当：**
- arXiv API 客户端能以正确限流抓取论文
- PDF 下载与缓存可靠工作
- Docling 解析器能从科学论文抽取结构化内容
- 数据库存储完整论文元数据及其关系
- 完整管道能端到端处理论文并带错误处理
- Airflow DAG 已配置并可供自动化
- 所有组件都展示生产级错误处理与监控
