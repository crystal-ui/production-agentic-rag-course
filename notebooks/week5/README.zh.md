# 第 5 周：接入 LLM 的完整 RAG 系统

> 本文为 [README.md](./README.md) 的一对一中文译本。

## 概览

第 5 周通过把 Ollama LLM 与混合检索集成，完成我们的**生产级 RAG 系统**。系统实现 **6 倍更快的性能**（120s → 15–20s）、实时流式输出，并包含 Gradio Web 界面。

## 我们构建了什么

- **本地 LLM 集成**：带 llama3.2 模型的 Ollama 服务
- **性能优化**：prompt 缩短 80%，速度提升 6 倍
- **流式 API**：通过 Server-Sent Events 实时响应
- **Gradio 界面**：支持流式的交互式 Web UI
- **生产就绪**：清晰 API 设计，两个聚焦端点

## 架构

<p align="center">
  <img src="../../static/week5_rag_architecture.png" alt="第 5 周完整 RAG 系统架构" width="900">
  <br>
  <em>完整 RAG 系统：LLM 生成层（Ollama）、混合检索管道与 Gradio 界面</em>
</p>

## 快速开始

### 1. 启动服务
```bash
docker compose up --build -d
```

### 2. 测试 RAG 端点
```bash
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "What are transformers?", "top_k": 3, "use_hybrid": true}'
```

### 3. 启动 Gradio 界面
```bash
uv run python gradio_launcher.py
# Open http://localhost:7861
```

## API 端点

### 标准 RAG - `/api/v1/ask`
- **用途**：带元数据的完整响应
- **响应时间**：15–20 秒
- **适用场景**：批处理、API 集成

### 流式 RAG - `/api/v1/stream`
- **用途**：实时 token 生成
- **首 token 时间**：2–3 秒
- **适用场景**：交互式 UI、更好的 UX

### 请求格式
```json
{
    "query": "Your question",
    "top_k": 3,              // Chunks to retrieve (1-10)
    "use_hybrid": true,      // BM25 + vector search
    "model": "llama3.2:1b",  // LLM model
    "categories": ["cs.AI"]  // Optional filter
}
```

## 性能

| 配置 | 响应时间 | 适用场景 |
|--------------|---------------|----------|
| `top_k=1, BM25` | ~2.4s | 快速回答 |
| `top_k=3, Hybrid` | ~15-20s | 质量平衡 |
| `top_k=5, Hybrid` | ~25-30s | 更全面 |

**关键优化**：
- 去掉冗余元数据（prompt 缩短 80%）
- 共享代码架构（DRY 原则）
- 300 词响应上限，答案更聚焦
- 自动来源去重

## 配置

```bash
# .env file
OLLAMA_HOST=http://ollama:11434
OLLAMA__DEFAULT_MODEL=llama3.2:1b
JINA_API_KEY=your_key_here  # For embeddings
```

## 测试

### 运行 Notebook
```bash
jupyter notebook notebooks/week5/week5_complete_rag_system.ipynb
```

### 测试流式
```bash
curl -X POST "http://localhost:8000/api/v1/stream" \
  -H "Content-Type: application/json" \
  -d '{"query": "Explain attention mechanism", "top_k": 2}' \
  --no-buffer
```

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| `/stream` 返回 404 | 重建 API：`docker compose build api && docker compose restart api` |
| 响应慢 | 用更小模型：`llama3.2:1b`，或降低 `top_k` |
| 没有 Gradio | 端口改为 7861：`http://localhost:7861` |
| Ollama 报错 | 检查服务：`docker exec rag-ollama ollama list` |

## 项目结构

```
src/
├── routers/
│   └── ask.py              # RAG endpoints
├── services/
│   └── ollama/
│       ├── client.py       # LLM client
│       └── prompts/        # System prompts
├── gradio_app.py           # Web interface
└── gradio_launcher.py      # Launcher script

notebooks/week5/
├── README.md               # This file
└── week5_complete_rag_system.ipynb
```

## 下一步

- **增强**：加入对话记忆、反馈循环
- **优化**：实现缓存、微调模型
- **部署**：加入鉴权、监控、负载均衡

## 资源

- [Notebook 教程](./week5_complete_rag_system.ipynb)
- [API 文档](http://localhost:8000/docs)
- [Gradio 界面](http://localhost:7861)
- [Ollama 模型](https://ollama.ai/library)
