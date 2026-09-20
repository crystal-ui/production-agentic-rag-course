# 第 6 周：用 Langfuse 与 Redis 做生产监控和缓存

> 本文为 [README.md](./README.md) 的一对一中文译本。

## 概览

第 6 周为我们的 RAG 系统加入生产级监控与智能缓存。我们集成 Langfuse 做完整管道可观测性，用 Redis 做高性能响应缓存。

## 我们构建了什么

- **Langfuse 集成**：端到端 RAG 流水线 tracing 与分析
- **Redis 缓存**：重复查询快 150–400 倍
- **性能监控**：实时指标与系统健康
- **生产就绪**：企业级可观测性与优化

## 架构

<p align="center">
  <img src="../../static/week6_monitoring_and_caching.png" alt="第 6 周监控与缓存架构" width="900">
  <br>
  <em>第 6 周架构：Langfuse tracing 与 Redis 缓存集成</em>
</p>

### 数据流
```
Query → Cache Check → [Hit: ~100ms] | [Miss: Full Pipeline ~15s] → Cache Store → Langfuse Trace
```

## 关键特性

### **Langfuse 可观测性**
- 完整 RAG 流水线 tracing，含性能分解
- 用户分析、查询模式与成功率追踪
- 带成本与用量指标的实时监控 dashboard
- 质量洞察：答案相关性与来源归属

### **Redis 智能缓存**
- **精确匹配策略**：感知参数的 cache key，精确匹配
- **性能**：重复查询快 150–400 倍（约 100ms vs 15–20s）
- **TTL 管理**：默认 24 小时过期，可配置
- **未来增强**：可升级为语义相似缓存以支持模糊匹配

## 快速开始

### 环境设置
```bash
# Required environment variables
LANGFUSE__SECRET_KEY=sk_lf_your_secret_key
LANGFUSE__PUBLIC_KEY=pk_lf_your_public_key
REDIS__HOST=redis
REDIS__TTL_HOURS=24
```

### 启动服务
```bash
docker compose up --build -d
```

### 测试缓存性能
```bash
# First request (cache miss ~15-20s)
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "What are transformers?", "top_k": 3}'

# Second identical request (cache hit ~100ms)
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "What are transformers?", "top_k": 3}'
```

## 性能基准

| 场景 | 响应时间 | 提升 |
|----------|---------------|-------------|
| **缓存未命中** | 15–20 秒 | 基线 |
| **缓存命中** | 50–100ms | **快 150–400 倍** |
| **监控开销** | <2% | 影响可忽略 |

## 测试

### 运行 Notebook
```bash
jupyter notebook notebooks/week6/week6_cache_testing.ipynb
```

### 监控系统健康
```bash
# Check Redis connectivity
redis-cli ping

# View cache statistics  
curl "http://localhost:8000/api/v1/health"

# Access Langfuse dashboard
# Visit: https://cloud.langfuse.com (or your self-hosted instance)
```

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| **缓存不工作** | 检查 Redis：`redis-cli ping` |
| **没有 Langfuse traces** | 验证环境变量：`LANGFUSE__*` |
| **响应慢** | 监控缓存命中率与系统资源 |

## 下一步

- **增强缓存**：升级为语义相似缓存以支持模糊匹配
- **高级分析**：自定义 dashboard 与 A/B 测试框架
- **生产扩展**：分布式缓存与自动监控
- **质量优化**：用户反馈集成与答案打分

## 资源

- **Notebook**：[week6_cache_testing.ipynb](./week6_cache_testing.ipynb)
- **Langfuse Dashboard**：https://cloud.langfuse.com
- **Redis 文档**：https://redis.io/docs

---

第 6 周把你的 RAG 系统变成生产级服务：性能提升 150–400 倍，并具备全面可观测性。
