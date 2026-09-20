# 第 3 周：先做关键词检索——关键基础

> 本文为 [README.md](./README.md) 的一对一中文译本。

> **🚨 90% 问题：** 大多数 RAG 系统一上来就做向量检索，错过了支撑最强检索系统的基础。我们要做对。

本目录包含 arXiv Paper Curator 项目第 3 周材料：我们用 OpenSearch 和 BM25 打分，实现专业 RAG 系统所依赖的**关键词检索基础**。

## 🎯 为什么先做关键词检索？

**专业路径：** 不同于一上来就做向量 embedding 的教程，我们先打成功公司使用的基础：

1. **🔍 精确匹配能力：** 关键词擅长找特定技术术语、论文 ID 和精确短语
2. **📊 结果可解释：** 你能准确理解一篇论文为什么被召回
3. **⚡ 速度与效率：** BM25 计算快，不需要昂贵的 embedding 模型
4. **📈 生产现实：** Elasticsearch、Algolia 以及企业搜索都以关键词检索为根基

**学习路径：**
```
Week 3: Master BM25 keyword search    ← YOU ARE HERE
Week 4: Add intelligent chunking
Week 5: Introduce vector embeddings for hybrid retrieval  
Week 6: Optimize the complete system
```

## 🚀 开始之前

**必要环境设置：**
```bash
# 1. Ensure you have the correct environment configuration
cp .env.example .env

# 2. Verify OpenSearch settings are properly configured
# Your .env should contain these critical Week 3 settings:
# OPENSEARCH__HOST=http://opensearch:9200
# OPENSEARCH__INDEX_NAME=arxiv-papers
```

**重要：** 第 3 周需要正确配置 `.env`，以便 OpenSearch 连通与建索引。`.env.example` 中的默认值开箱即用。

## 内容

### `week3_opensearch.ipynb`
一份完整的 Jupyter notebook，引导你构建关键词检索基础：

1. **基础设施校验**
   - 确认第 1–2 周全部服务运行正常
   - OpenSearch 集群健康与连通性测试
   - 第 3 周检索组件的环境准备

2. **OpenSearch 服务集成**
   - 用工厂模式构建生产级检索客户端
   - 用 FastAPI lifespan 管理做依赖注入
   - 创建带合适 analyzer 的 JSON 索引配置

## 🔍 OpenSearch Dashboard 实战

*你可以在这里加入自己的 OpenSearch dashboard 截图，展示带 BM25 打分与结果排序的真实论文检索。*

<p align="center">
  <img src="../../static/week3_opensearch_dashboard.png" alt="OpenSearch Dashboard 检索论文" width="800">
  <br>
  <em>OpenSearch Dashboards 展示带 BM25 相关性打分的关键词检索结果</em>
</p>

**检索架构概览：**
- **OpenSearch Client**：工厂模式服务，含健康监控与 CRUD
- **Query Builder**：高级检索查询构建，含字段 boosting 与过滤
- **Index Management**：基于 JSON 的 schema，含英语 analyzer 与严格 mapping
- **Search API**：RESTful 端点（`/search`），GET/POST 对应不同查询类型
- **BM25 Algorithm**：工业标准文本相关性打分，支持多字段检索

3. **索引管理与 Schema 设计**
   - 为学术论文创建带正确字段 mapping 的索引
   - 实现英语 analyzer 以提升检索相关性
   - 测试索引健康、统计与文档管理操作

4. **文档索引管道**
   - 把论文数据从 PostgreSQL 迁到 OpenSearch
   - 带错误处理与校验的批量索引
   - 实时索引验证与性能监控

5. **BM25 检索实现**
   - 跨 title、abstract 与 content 的多字段检索，带字段 boosting
   - 高级查询特性：高亮、分页、类别过滤
   - 按特别要求测试两字母查询（AI、ML、NN、CV）
   - 模糊匹配与拼写容错配置

6. **Search API 开发**
   - FastAPI 与 OpenSearch 依赖注入集成
   - RESTful 端点：`GET /search` 用于简单查询，`POST /search` 用于高级检索
   - 带分页、元数据与高亮片段的响应格式
   - 错误处理与服务健康监控端点

7. **Airflow 管道更新**
   - 顺序任务执行：setup → fetch → opensearch → report → cleanup
   - 用真实 OpenSearch 索引替换占位操作
   - 更新每日报告，加入检索统计与健康指标
   - 优雅处理 PDF 处理限制（>20MB 或 >30 页）

8. **端到端管道验证**
   - 完整流程测试：arXiv API → PostgreSQL → OpenSearch → Search API
   - 性能基准与响应时间测量
   - 带监控与告警的生产就绪评估

**第 3 周架构：**

<p align="center">
  <img src="../../static/week3_opensearch_flow.png" alt="第 3 周 OpenSearch 流程架构" width="800">
  <br>
  <em>完整第 3 周架构，展示 OpenSearch 集成流程</em>
</p>


## 已实现的关键特性

### 🔍 **生产级检索系统**
- **BM25 Scoring**：工业标准相关性排序算法
- **多字段检索**：查询 title（3x boost）、abstract（2x boost）和 content（1x boost）
- **高级过滤**：类别过滤、日期范围与自定义字段查询
- **实时高亮**：结果中用 HTML 标记高亮检索词
- **分页支持**：用 size/from 参数高效处理大结果集

### 🏗️ **清晰架构实现**
- **工厂模式**：所有组件一致的服务创建方式
- **依赖注入**：用 FastAPI lifespan 做正确的服务生命周期管理
- **Query Builder 模式**：灵活构建复杂检索查询
- **错误处理**：全面异常管理与优雅降级

### 📊 **性能与监控**
- **亚 100ms 检索**：为实时应用优化的查询性能
- **健康监控**：集群状态、索引统计与服务可用性
- **资源限制**：PDF 处理约束以防内存问题
- **可扩展性**：支持水平扩展的设计模式

### 🔧 **开发最佳实践**
- **类型安全**：全代码库完整类型标注
- **测试**：用真实数据场景做端到端验证
- **文档**：完整 docstring 与架构说明
- **代码质量**：沿用第 1–2 周已建立的模式

## 预期成果

完成第 3 周后，学员将拥有：

1. **可用的检索系统**：功能完整的 OpenSearch BM25 检索，索引 28+ 篇论文
2. **RESTful API**：同时支持简单与高级查询的完整检索 API
3. **生产架构**：遵循行业最佳实践的清晰、可扩展代码
4. **性能洞察**：理解检索优化与监控策略
5. **端到端管道**：为第 4 周开发准备好的完整 RAG 系统基础

## 成功标准

- ✅ **OpenSearch 集成**：工厂模式服务与健康检查可用
- ✅ **BM25 检索**：多字段查询的相关性打分能返回相关结果
- ✅ **高级特性**：过滤、高亮、分页与模糊检索可用
- ✅ **API 端点**：GET 与 POST 的 RESTful 检索可用
- ✅ **两字母查询**：按特别要求，AI、ML、NN、CV 查询可用
- ✅ **管道集成**：Airflow DAG 已更新以支持 OpenSearch 索引
- ✅ **性能**：亚 100ms 检索响应，并有正确错误处理

## 下一步？

**第 4 周预告**：切块策略与混合检索
- 更好保留上下文的智能文档切块策略
- 把关键词检索（BM25）与向量 embedding 结合做混合检索
- 为现代 RAG 系统打下完整基础
- 用评估指标衡量检索质量

---

**准备好构建生产级检索了吗？我们从第 3 周开始！** 🔍✨
