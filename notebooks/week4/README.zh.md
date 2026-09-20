# 第 4 周：文档切块与混合检索

> 本文为 [README.md](./README.md) 的一对一中文译本。

## 概览

第 4 周实现一套**生产级混合检索系统**，把 BM25 关键词检索的精确性与向量 embedding 的语义理解结合起来。该系统通过把文档智能切成可检索 chunk，并支持多种检索模式，为检索增强生成（RAG）打下基础。

## 我们构建了什么

### 🧩 **基于章节的文档切块**
- **智能分段**：利用文档结构（已解析章节）作为自然切块边界
- **上下文保留**：chunk 之间 100 词 overlap，维持语义连续性
- **自适应处理**：同时处理结构化（有章节）与非结构化（按段落）文档
- **最优大小**：目标 600 词/chunk，最小阈值 100 词

### 🔍 **统一混合检索系统**
- **单索引架构**：一个 OpenSearch 索引（`arxiv-papers-chunks`）支持全部检索模式
- **多种检索类型**：
  - **BM25 关键词检索**：快速（约 50ms）的传统文本匹配
  - **向量相似度检索**：使用 1024 维 embedding 的语义检索
  - **混合检索**：用 RRF（Reciprocal Rank Fusion）结合两种方法
- **生产 API**：RESTful 端点 `/api/v1/hybrid-search/`，带全面校验

### 🤖 **真实 Embedding 集成**
- **Jina AI Embeddings**：面向检索优化的生产级 1024 维向量
- **自动生成**：FastAPI 端点自动生成查询 embedding
- **回退策略**：embedding 不可用时优雅降级到 BM25
- **性能优化**：高效的 embedding 生成与存储

## 架构

### 系统概览

<p align="center">
  <img src="../../static/week4_hybrid_opensearch.png" alt="第 4 周混合检索架构" width="800">
  <br>
  <em>完整第 4 周架构：切块、embedding 与 RRF 融合的混合检索</em>
</p>

### 数据流
```
Raw Papers → PDF Parsing → Section Extraction → Chunking → Embedding → Indexing → Search
```

上图展示完整第 4 周实现：
- **数据处理管道**：arXiv 论文流经切块与 embedding 生成
- **统一 OpenSearch 索引**：单一索引支持 BM25、向量与混合检索模式
- **混合检索管道**：RRF 融合，结合关键词精确性与语义理解
- **生产 API 层**：带自动 embedding 生成的 FastAPI 端点

### 系统组件

#### **1. 文档处理管道**
```python
# Located in: src/services/indexing/text_chunker.py
TextChunker.chunk_paper(
    title="Paper Title",
    abstract="Abstract text",
    full_text="Complete paper content",
    sections=parsed_sections_dict,
    target_words=600,
    overlap_words=100
)
```

#### **2. Embedding 服务**
```python
# Located in: src/services/embeddings/factory.py
embeddings_service = make_embeddings_service()
vectors = await embeddings_service.embed_query(["query text"])
```

#### **3. 统一检索客户端**
```python
# Located in: src/services/opensearch/client.py
results = opensearch_client.search_unified(
    query="machine learning",
    query_embedding=vector,
    use_hybrid=True,
    size=10
)
```

#### **4. 生产 API**
```python
# Located in: src/routers/hybrid_search.py
POST /api/v1/hybrid-search/
{
  "query": "neural networks",
  "use_hybrid": true,
  "size": 5
}
```

## 关键特性

### **混合检索模式**

| 模式 | 速度 | 召回 | 精确 | 适用场景 |
|------|--------|--------|-----------|----------|
| **仅 BM25** | ~50ms | 高 | 中 | 精确关键词匹配 |
| **仅向量** | ~100ms | 中 | 高 | 语义相似 |
| **混合（RRF）** | ~2-4s | 高 | 高 | 整体相关性最好 |

### **RRF（Reciprocal Rank Fusion）**
- **算法**：用倒数排名融合结合 BM25 与向量检索排序
- **实现**：手动融合算法（兼容 OpenSearch 2.19）
- **加权**：关键词相关性与语义相关性之间可配置平衡
- **回退**：向量检索失败时自动回退到 BM25

### **基于章节的切块策略**

```python
# Chunking Parameters (optimized through testing)
CHUNK_SIZE = 600        # Target words per chunk
OVERLAP_SIZE = 100      # Words overlapping between chunks  
MIN_CHUNK_SIZE = 100    # Minimum viable chunk size
SECTION_BASED = True    # Use document structure when available
```

**好处**：
- **语义连贯**：chunk 尊重文档的自然边界
- **上下文保留**：overlap 防止边界处信息丢失
- **检索准确**：用户查询能更好匹配到相关内容
- **可扩展**：可处理从 1,000 到 100,000+ 词的文档

## 实现细节

### **OpenSearch 索引配置**

**索引名**：`arxiv-papers-chunks`

**关键字段**：
```json
{
  "arxiv_id": "2508.18563v1",
  "title": "Paper title",
  "chunk_text": "Chunk content...",
  "chunk_id": "unique_chunk_identifier", 
  "section_name": "Introduction",
  "embedding": [0.123, 0.456, ...],  // 1024 dimensions
  "paper_categories": ["cs.AI", "cs.LG"],
  "published_date": "2025-08-25T23:43:33"
}
```

### **检索查询结构**

**BM25 查询**（关键词匹配）：
```json
{
  "query": {
    "bool": {
      "should": [
        {"match": {"chunk_text": {"query": "machine learning", "fuzziness": "AUTO"}}},
        {"match": {"title": {"query": "machine learning", "boost": 2.0}}},
        {"match": {"abstract": {"query": "machine learning", "boost": 1.5}}}
      ]
    }
  }
}
```

**混合查询**（RRF 手动融合）：
1. 执行 BM25 查询 → 得到排序结果
2. 执行向量查询 → 得到排序结果
3. 应用 RRF 融合算法 → 合并排序
4. 返回带混合分数的合并结果

### **环境配置**

**必需变量**：
```bash
# Core Services
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
OPENSEARCH__HOST=http://opensearch:9200

# Embeddings (Required for Hybrid Search)
JINA_API_KEY=jina_your_api_key_here

# Chunking Configuration
CHUNKING__CHUNK_SIZE=600
CHUNKING__OVERLAP_SIZE=100
CHUNKING__MIN_CHUNK_SIZE=100
CHUNKING__SECTION_BASED=true

# OpenSearch Configuration
OPENSEARCH__INDEX_NAME=arxiv-papers
OPENSEARCH__CHUNK_INDEX_SUFFIX=chunks
OPENSEARCH__VECTOR_DIMENSION=1024
```

## API 参考

### **混合检索端点**

**端点**：`POST /api/v1/hybrid-search/`

**请求体**：
```json
{
  "query": "transformer neural networks",
  "use_hybrid": true,
  "size": 10,
  "from": 0,
  "categories": ["cs.AI", "cs.LG"],
  "latest_papers": false,
  "min_score": 0.0
}
```

**响应**：
```json
{
  "query": "transformer neural networks",
  "total": 15,
  "hits": [
    {
      "arxiv_id": "2508.18563v1",
      "title": "Paper Title",
      "authors": "Author Names",
      "abstract": "Paper abstract...",
      "score": 0.8542,
      "chunk_text": "Relevant chunk content...",
      "chunk_id": "chunk_uuid",
      "section_name": "Related Work"
    }
  ],
  "size": 10,
  "from": 0,
  "search_mode": "hybrid"
}
```


## 性能基准

**测试环境**：3 篇论文、81 个 chunk、单节点 OpenSearch

| 检索类型 | 平均响应时间 | 吞吐量 | Recall@10 | Precision@10 |
|-------------|-------------------|------------|-----------|--------------|
| 仅 BM25 | 52ms | ~200 req/s | 0.78 | 0.65 |
| 仅向量 | 105ms | ~95 req/s | 0.82 | 0.71 |
| 混合（RRF） | 2.4s | ~25 req/s | 0.89 | 0.84 |

**关键洞察**：
- **混合检索**在牺牲响应时间的前提下提供最好相关性
- **BM25** 非常适合高吞吐关键词匹配
- **向量检索**语义理解好、速度中等
- **Embedding 生成**约占混合检索时间的 ~2s

## 生产部署

### **扩展考量**

**OpenSearch 集群**：
```yaml
# Recommended minimum for production
opensearch:
  image: opensearchproject/opensearch:2.19.0
  environment:
    - cluster.name=rag-cluster
    - node.name=rag-node-1
    - discovery.type=single-node
    - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
  deploy:
    resources:
      limits:
        memory: 2G
      reservations:
        memory: 1G
```

**Embedding 服务优化**：
- **批处理**：每次 API 调用处理多个 embedding
- **缓存**：缓存频繁查询的 embedding
- **限流**：遵守 Jina AI API 限制（1000 请求/分钟）
- **回退策略**：embedding 不可用时仅用 BM25

### **监控与可观测性**

**关键指标**：
- 检索请求延迟（p50、p95、p99）
- Embedding 生成成功率
- 索引文档数量与大小
- 检索模式使用分布（BM25 vs Hybrid）

**健康检查**：
- OpenSearch 集群健康
- Embedding 服务可用性
- 索引文档数量校验
- 样例查询执行

## 故障排除

### **常见问题**

**1. 混合检索实际返回 BM25 模式**
```bash
# Check embedding service
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "test", "use_hybrid": true}'

# Check logs for embedding errors
docker compose logs api | grep -i embedding
```

**2. 检索结果为空**
```bash
# Verify index exists and has documents
curl "http://localhost:9200/arxiv-papers-chunks/_count"

# Check index mapping
curl "http://localhost:9200/arxiv-papers-chunks/_mapping"
```

**3. Embedding 生成缓慢**
```bash
# Check Jina API key configuration
docker compose exec api env | grep JINA

# Test direct embedding service
curl -X POST "https://api.jina.ai/v1/embeddings" \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -d '{"model": "jina-embeddings-v3", "input": ["test"]}'
```

## 测试

### **运行第 4 周 Notebook**

1. **启动服务**：
```bash
docker compose up --build -d
```

2. **打开 Notebook**：
```bash
cd notebooks/week4
jupyter notebook week4_hybrid_search.ipynb
```

3. **执行全部 cell**：notebook 包含：
   - 环境搭建与健康检查
   - 基于章节的切块演示
   - 用 Jina AI 做真实 embedding 生成
   - 全部检索模式（BM25、向量、混合）
   - 生产 API 端点测试
   - 性能对比

### **手工测试**

**测试 BM25 检索**：
```bash
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "machine learning", "use_hybrid": false, "size": 3}'
```

**测试混合检索**：
```bash
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "neural networks", "use_hybrid": true, "size": 3}'
```

## 下一步（第 5 周）

第 4 周为第 5 周的 LLM 集成提供检索基础：

1. **LLM 集成**：接入 Ollama 做答案生成
2. **RAG 流水线**：查询 → 检索 → 上下文 → 生成 → 响应
3. **上下文管理**：为 LLM 输入优化召回的 chunk
4. **答案质量**：实现引用与来源归属
5. **对话记忆**：支持多轮对话

混合检索系统已**生产就绪**，能为高质量 RAG 应用提供所需的检索准确度。

## 文件结构

```
src/
├── routers/
│   └── hybrid_search.py          # FastAPI endpoints
├── services/
│   ├── opensearch/
│   │   ├── client.py              # Unified search client
│   │   ├── factory.py             # Client factory
│   │   └── index_config_hybrid.py # Index configuration
│   ├── indexing/
│   │   ├── text_chunker.py        # Section-based chunking
│   │   ├── hybrid_indexer.py      # Document indexing
│   │   └── factory.py             # Indexing service factory
│   └── embeddings/
│       ├── jina_client.py         # Jina AI client
│       └── factory.py             # Embedding service factory
├── schemas/
│   └── api/
│       └── search.py              # Request/response models
└── config.py                      # Configuration management

notebooks/week4/
├── README.md                      # This document
├── week4_hybrid_search.ipynb      # Interactive tutorial
└── data/                          # Sample data directory
```

## 资源

- **OpenSearch 文档**：https://opensearch.org/docs/
- **Jina AI Embeddings**：https://jina.ai/embeddings/
- **FastAPI 文档**：https://fastapi.tiangolo.com/
- **Reciprocal Rank Fusion 论文**：https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf
- **第 4 周 Notebook**：[week4_hybrid_search.ipynb](./week4_hybrid_search.ipynb)
