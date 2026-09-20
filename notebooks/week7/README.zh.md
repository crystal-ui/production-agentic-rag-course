# 第 7 周：LangGraph Agentic RAG + Telegram Bot

> 本文为 [README.md](./README.md) 的一对一中文译本。

## 概览

第 7 周为 arXiv Paper Curator 增加两大增强：

1. **🤖 基于 LangGraph 的 Agentic RAG** - 带决策能力的智能、自适应检索
2. **💬 Telegram Bot 集成** - 可在手机/桌面使用的对话式界面

---

## 🧠 第 1 部分：基于 LangGraph 的 Agentic RAG

### 什么是 Agentic RAG？

**传统 RAG**（第 5–6 周）：
```
Query → Always Retrieve → Generate Answer
```

**Agentic RAG**（第 7 周）：
```
Query → Agent Decides:
  ├─ Simple question? → Respond directly (faster!)
  └─ Research needed? → Retrieve
       ├─ Relevant docs? → Generate answer
       └─ Not relevant? → Rewrite query → Try again
```

### 关键特性

- **🎯 智能决策** - LLM 决定何时真正需要检索
- **📊 文档打分** - 验证召回的论文是否相关
- **🔄 查询优化** - 改写含糊查询以获得更好结果
- **🔍 推理透明** - 展示 agent 的决策步骤
- **♻️ 迭代改进** - 必要时用更好的查询重试

### 我们构建了什么

```
src/services/agents/
├── tools.py            # Retriever tool wrapping OpenSearch
├── nodes.py            # 4 graph nodes (query, grade, rewrite, generate)
├── agentic_rag.py      # LangGraph workflow + service
├── prompts.py          # LLM prompt templates
└── factory.py          # Dependency injection

src/routers/
└── agentic_ask.py      # FastAPI endpoint

Total: ~750 LOC following SOLID, KISS, DRY principles
```

### 架构

```
LangGraph Workflow:

START
  ↓
generate_query_or_respond
  ├─ No retrieval needed → END (direct response)
  └─ Needs retrieval → retrieve (ToolNode)
       ↓
     grade_documents
       ├─ Relevant → generate_answer → END
       └─ Not relevant → rewrite_query → (loop back)
```

### 新 API 端点

**`POST /api/v1/ask-agentic`**

```json
// Request
{
  "query": "What are transformers in ML?",
  "top_k": 3,
  "use_hybrid": true
}

// Response
{
  "query": "What are transformers in ML?",
  "answer": "Transformers are neural network architectures...",
  "sources": ["https://arxiv.org/pdf/1706.03762.pdf"],
  "chunks_used": 3,
  "search_mode": "hybrid",
  "reasoning_steps": [
    "Decided to retrieve relevant papers",
    "Retrieved documents from database",
    "Generated answer from relevant documents"
  ],
  "retrieval_attempts": 1
}
```

### 快速开始：Agentic RAG

**1. 确认服务在运行：**
```bash
docker compose up --build -d
```

**2. 用 cURL 测试：**
```bash
# Simple question (should respond directly)
curl -X POST http://localhost:8000/api/v1/ask-agentic \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is 2+2?",
    "top_k": 3,
    "use_hybrid": true
  }'

# Research question (should retrieve papers)
curl -X POST http://localhost:8000/api/v1/ask-agentic \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are attention mechanisms?",
    "top_k": 3,
    "use_hybrid": true
  }'
```

**3. 交互式测试：**
```bash
# Open Jupyter notebook
jupyter notebook notebooks/week7/week7_agentic_rag.ipynb
```

### 对比：传统 vs Agentic

| 特性 | 传统 RAG | Agentic RAG |
|---------|----------------|-------------|
| **检索** | 总是检索 | 按需决定 |
| **相关性检查** | 无 | 对文档打分 |
| **查询优化** | 无 | 必要时改写 |
| **迭代** | 单次 | 多次尝试 |
| **透明度** | 黑盒 | 展示推理 |
| **简单问题** | ~15-20s | ~2-5s（不检索） |
| **复杂问题** | 单次尝试 | 迭代优化 |

### 测试场景

**场景 1：直接回答（不检索）**
- 查询："What is 5 + 7?"
- 预期：Agent 回答 "12"，不检索论文
- 推理："Responded directly without retrieval"

**场景 2：检索成功**
- 查询："What are transformers in machine learning?"
- 预期：Agent 检索论文、判定相关、生成答案
- 推理："Decided to retrieve" → "Retrieved documents" → "Generated answer"

**场景 3：查询改写**
- 查询："Tell me about ML stuff"（含糊）
- 预期：Agent 检索、判定不相关、改写查询、再试
- 推理："Retrieved" → "Not relevant" → "Rewritten query" → "Retrieved again" → "Generated answer"

### 遵循的设计原则

- ✅ **SOLID** - 单一职责、依赖倒置、组合
- ✅ **KISS** - 简单节点（<30 行）、清晰逻辑
- ✅ **DRY** - 复用已有服务（OpenSearch、Ollama、Jina）
- ✅ **YAGNI** - 只实现需要的部分
- ✅ **Explicit** - 类型标注、docstring、清晰命名
- ✅ **2025 最佳实践** - MessagesState、ToolNode、tools_condition

### 文档

- **实现计划**：`docs/AGENTIC_RAG_IMPLEMENTATION_PLAN.md`
- **测试计划**：`docs/AGENTIC_RAG_TESTING_PLAN.md`
- **LangGraph 2025 模式**：`docs/LANGGRAPH_2025_BEST_PRACTICES.md`
- **设计原则**：`docs/DESIGN_PRINCIPLES.md`
- **交互式 Notebook**：`notebooks/week7/week7_agentic_rag.ipynb`

---

## 💬 第 2 部分：Telegram Bot 集成

### 我们构建了什么

- **🤖 完整 Telegram Bot 集成**：带命令支持的对话式界面
- **💬 自然语言提问**：用日常语言提问，获得带来源的答案
- **⚡ 第 6 周全部特性**：Redis 缓存（150–400 倍加速）与 Langfuse tracing
- **🎯 交互命令**：`/start`、`/help`、`/ask`、`/search`、`/settings`、`/status`
- **👤 用户会话管理**：按用户保存偏好与对话历史
- **📱 移动优先**：丰富消息格式，arXiv 链接可点击
- **🔐 可选访问控制**：需要时可白名单特定用户

## 架构

### 数据流
```
Telegram User
    ↓
Telegram Bot (Polling/Webhook)
    ↓
TelegramService + Handlers
    ↓ [Langfuse Tracing]
Cache Check (Redis)
    ├─ Hit → Instant Response (~100ms)
    └─ Miss → Full RAG Pipeline
        ↓
Hybrid Search (OpenSearch BM25 + Vector)
        ↓
LLM Generation (Ollama)
        ↓
Cache Store (Redis)
        ↓
Format Response → Send to Telegram
```

### 新组件

```
src/services/telegram/
├── client.py           # Main Telegram bot service
├── handlers.py         # Command and message handlers
├── formatters.py       # Message formatting utilities
├── keyboards.py        # Interactive inline keyboards
├── user_manager.py     # User settings and sessions
└── factory.py          # Factory function

src/schemas/telegram/
├── messages.py         # Message validation schemas
├── commands.py         # Command schemas
└── user_settings.py    # User preferences schema
```

## 关键特性

### **对话式界面**
- 自然语言提问：直接发消息即可
- 带历史追踪的上下文对话
- 自动路由查询（命令 vs 普通消息）

### **丰富命令**
```
/start    - Welcome message with bot capabilities
/help     - Detailed usage instructions
/ask      - Explicitly ask a research question
/search   - Quick paper search by keywords
/settings - Customize preferences (search mode, results count)
/status   - Check system health and statistics
/clear    - Clear conversation history
```

### **交互式设置**
- 检索模式：Hybrid（BM25 + 语义）或仅 BM25
- 每次查询结果数：3、5 或 10 篇
- 类别过滤：All、cs.AI、cs.LG、cs.CL 等
- 模型选择：选择 LLM 模型
- 开关：流式、显示来源、紧凑模式

### **用户体验**
- 处理过程中的**正在输入指示**
- 支持 Markdown 的**富文本格式**
- arXiv 论文的**可点击链接**
- 长回复**自动拆分消息**
- 带论文元数据的**来源引用**
- 即时回复的**缓存指示**（⚡）

## 快速开始

### 前置条件

1. **Telegram 账号** - 在手机或电脑上安装 Telegram
2. **第 1–6 周全部服务在运行** - 完整 RAG 栈必须可用

### 步骤 1：创建你的 Telegram Bot

1. **打开 Telegram** 并搜索 `@BotFather`
2. **发送** `/newbot` 给 BotFather
3. **按提示操作**：
   - 选择名称（例如 "My arXiv Curator"）
   - 选择用户名（例如 "my_arxiv_curator_bot" - 必须以 "bot" 结尾）
4. **复制 bot token** - 你会收到类似：
   ```
   1234567890:ABCdefGHIjklMNOpqrsTUVwxyz-1234567
   ```

### 步骤 2：配置环境

把这些变量加入 `.env` 文件：

```bash
# Enable Telegram bot
TELEGRAM__ENABLED=true
TELEGRAM__BOT_TOKEN=your_token_from_botfather_here

# Optional: Restrict to specific users (comma-separated Telegram user IDs)
# Leave empty to allow all users
TELEGRAM__ALLOWED_USER_IDS=

# Use polling mode for development (webhook requires HTTPS)
TELEGRAM__USE_WEBHOOK=false
```

### 步骤 3：安装依赖

```bash
# Install python-telegram-bot library
uv sync
```

### 步骤 4：启动服务

```bash
# Start all services (includes Telegram bot)
docker compose up --build -d

# Check logs to verify Telegram bot started
docker compose logs -f api
```

你应该看到：
```
INFO - Telegram bot started successfully
INFO - Starting Telegram bot in polling mode
INFO - Bot commands set successfully
```

### 步骤 5：测试你的 Bot

1. **打开 Telegram** 并搜索你的 bot 用户名
2. **发送** `/start` 给你的 bot
3. **试着问**："What are transformers in machine learning?"
4. **确认**你收到带来源的答案！

## 测试说明

### 手工测试场景

#### 场景 1：基础命令

**测试 `/start` 命令：**
```
You: /start
Bot: 👋 Welcome to arXiv Paper Curator!
     [Shows capabilities and quick commands]
```

**测试 `/help` 命令：**
```
You: /help
Bot: 📚 arXiv Paper Curator Help
     [Shows detailed command documentation]
```

**测试 `/status` 命令：**
```
You: /status
Bot: 🔧 System Status
     ✅ OPENSEARCH
     ✅ OLLAMA
     ✅ CACHE
     [Shows system health]
```

#### 场景 2：RAG 问答

**测试简单问题：**
```
You: What are attention mechanisms?
Bot: [15-20s first time]
     *Answer:*
     Attention mechanisms allow models to dynamically focus on...

     📚 *Sources:*
     [1] *Attention Is All You Need*
         🔗 Read on arXiv - 1706.03762
         📊 Score: 12.456

     [2] *Neural Machine Translation...*
         🔗 Read on arXiv - 1409.0473
         📊 Score: 11.234

     ⚙️ Mode: hybrid
```

**测试缓存查询：**
```
You: What are attention mechanisms?
Bot: [~100ms second time ⚡]
     [Same answer as above]
     ⚙️ Mode: hybrid ⚡ Cached
```

#### 场景 3：Search 命令

**测试论文检索：**
```
You: /search transformer neural networks
Bot: 📖 Found 145 papers (showing top 10)

     1. *Attention Is All You Need*
        🔗 Read on arXiv - 1706.03762
        📊 Score: 12.456

     2. *BERT: Pre-training of Deep Bidirectional...*
        🔗 Read on arXiv - 1810.04805
        📊 Score: 11.234
     [...]
```

#### 场景 4：自定义设置

**测试 settings 命令：**
```
You: /settings
Bot: ⚙️ Your Settings

     *Search Mode:* HYBRID
     *Results per query:* 3 papers
     *Model:* llama3.2:1b
     *Categories:* All

     [Interactive buttons appear]:
     [🔍 Hybrid Search] [⚡ BM25 Only]
     [3 Results] [5 Results] [10 Results]
     [All Categories] [cs.AI] [cs.LG]
```

**测试更改设置：**
```
You: [Click "5 Results" button]
Bot: ✅ Results per query: 5

You: [Click "cs.AI" button]
Bot: ✅ Category filter: cs.AI
```

#### 场景 5：自然对话

**测试多轮对话：**
```
You: Tell me about BERT
Bot: [Provides answer about BERT with sources]

You: How does it differ from GPT?
Bot: [Answers about BERT vs GPT differences]

You: /clear
Bot: 🗑️ Conversation history cleared!
     Your settings have been preserved.
```

#### 场景 6：错误处理

**测试无效查询：**
```
You: asdfghjkl
Bot: ❌ No relevant papers found.
     Try different keywords or check your category filters.
```

**测试服务宕机时：**
```
You: test query
Bot: ❌ Error processing your question:
     [User-friendly error message]
```

### 验证清单

- [ ] **Bot 响应 `/start`** - 显示欢迎消息
- [ ] **Bot 响应全部命令** - `/help`、`/ask`、`/search`、`/settings`、`/status`、`/clear`
- [ ] **自然语言提问可用** - 不用命令也能提问
- [ ] **RAG 答案包含来源** - 引用论文并带 arXiv 链接
- [ ] **缓存可用** - 重复查询立即返回（⚡ 指示）
- [ ] **设置持久化** - 对话中更改会保留
- [ ] **交互按钮可用** - 可点击 inline keyboard 按钮
- [ ] **长回复正确拆分** - 消息不超过 Telegram 限制
- [ ] **Markdown 格式可用** - 粗体、斜体、链接正确渲染
- [ ] **Langfuse 显示 traces** - 在 http://localhost:3000 查看 Telegram 事件
- [ ] **正在输入指示显示** - 处理时出现 "Bot is typing..."
- [ ] **错误消息友好** - 不向用户暴露堆栈

### 性能测试

**缓存性能：**
```bash
# First query (cache miss)
You: What is machine learning?
Bot: [Response in ~15-20 seconds]

# Identical query (cache hit)
You: What is machine learning?
Bot: [Response in ~100ms ⚡]

# Verify 150-400x speedup!
```

**并发用户：**
```bash
# Test with multiple Telegram accounts
# Each user should have independent settings and sessions
```

**会话管理：**
```bash
# Stop using bot for 30 minutes (default timeout)
# Verify session cleanup happens automatically
```

## 配置

### 环境变量

```bash
# Enable/Disable Bot
TELEGRAM__ENABLED=true  # Set to false to disable

# Bot Token (Required)
TELEGRAM__BOT_TOKEN=your_token_here

# Deployment Mode
TELEGRAM__USE_WEBHOOK=false  # true for production
TELEGRAM__WEBHOOK_URL=https://your-domain.com
TELEGRAM__WEBHOOK_PATH=/telegram/webhook

# Access Control (Optional)
TELEGRAM__ALLOWED_USER_IDS=123456789,987654321  # Empty = allow all

# Behavior Settings
TELEGRAM__MAX_MESSAGE_LENGTH=4000  # Telegram limit is 4096
TELEGRAM__ENABLE_STREAMING=true
TELEGRAM__SESSION_TIMEOUT_MINUTES=30
TELEGRAM__RATE_LIMIT_MESSAGES_PER_MINUTE=20

# Default User Preferences
TELEGRAM__DEFAULT_TOP_K=3
TELEGRAM__DEFAULT_USE_HYBRID=true
TELEGRAM__DEFAULT_MODEL=llama3.2:1b
```

### 用户设置（可按用户自定义）

每个用户可以自定义：
- **检索模式**：Hybrid（BM25 + 语义）或仅 BM25
- **结果数量**：每次查询 3、5 或 10 篇
- **类别**：All、cs.AI、cs.LG、cs.CL、cs.CV、cs.NE
- **模型**：用于答案生成的 LLM 模型
- **显示**：显示来源、紧凑模式、流式

设置会跨会话保留，并存在内存中（生产环境可考虑 Redis/PostgreSQL 存储）。

## 架构细节

### 服务集成

Telegram bot 与第 1–6 周全部已有服务集成：

1. **OpenSearch** - 混合检索相关论文
2. **Jina Embeddings** - 语义检索能力
3. **Ollama LLM** - 答案生成
4. **Redis Cache** - 重复查询快 150–400 倍
5. **Langfuse** - Telegram 交互的完整 tracing
6. **PostgreSQL** - 论文元数据（经由 OpenSearch）

### 消息流

```python
# Simplified message flow
async def handle_message(update, context):
    1. Extract user_id, chat_id, text
    2. Check rate limits
    3. Get/create user settings
    4. Show "typing..." indicator
    5. Check cache (Redis)
    6. If cache hit → format and send
    7. If cache miss:
        a. Generate embedding (Jina)
        b. Search papers (OpenSearch)
        c. Build prompt with context
        d. Generate answer (Ollama)
        e. Cache result (Redis)
        f. Trace interaction (Langfuse)
    8. Format response (Markdown)
    9. Split if too long
    10. Send to Telegram
    11. Update conversation history
```

### 错误处理

Bot 实现优雅降级：

- **Markdown 格式失败** → 回退为纯文本
- **Embedding 生成失败** → 回退为 BM25 检索
- **缓存不可用** → 不使用缓存继续
- **Langfuse tracing 失败** → 打警告日志，继续
- **服务错误** → 对用户友好的错误消息

### 限流

内置限流防止滥用：
- **按用户限制**：20 条消息/分钟（可配置）
- **自动节流**：由 Telegram 库强制执行
- **会话超时**：30 分钟无活动

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| **Bot 不响应** | 检查 `TELEGRAM__ENABLED=true` 以及有效的 `BOT_TOKEN` |
| **"Unauthorized" 错误** | Bot token 无效，用 @BotFather 重新生成 |
| **Bot 有响应但没有答案** | 检查 OpenSearch、Ollama 和 embeddings 服务 |
| **响应慢** | 第一次查询总是慢，后续查询走缓存 |
| **Markdown 格式坏了** | Bot 会自动回退为纯文本 |
| **找不到 bot** | 确认 bot 用户名正确，并以 "bot" 结尾 |
| **"Forbidden" 错误** | 检查 `ALLOWED_USER_IDS` 中的 user_id 限制 |
| **内存问题** | 用户会话存在内存中，监控 RAM 用量 |

### 调试模式

启用详细日志：

```bash
# In .env
DEBUG=true

# Check bot logs
docker compose logs -f api | grep telegram
```

### 健康检查

```bash
# Check if bot is running
curl http://localhost:8000/api/v1/health

# Should show telegram_service status
```

### 常见修复

**Bot 起不来：**
```bash
# 1. Check token is valid
# 2. Restart API service
docker compose restart api

# 3. Check logs for errors
docker compose logs api
```

**响应太慢：**
```bash
# 1. Check cache is working
docker exec rag-redis redis-cli ping  # Should return PONG

# 2. Check cache hit rate
# Look for ⚡ indicator in bot responses

# 3. Verify Langfuse tracing not blocking
LANGFUSE__ENABLED=false  # Temporarily disable
```

## 性能基准

| 指标 | 数值 | 说明 |
|--------|-------|-------|
| **首次查询** | 15–20s | 完整 RAG 流水线执行 |
| **缓存查询** | 50–100ms | **快 150–400 倍** |
| **正在输入指示** | <500ms | 立即显示 |
| **仅检索** | 2–3s | `/search` 命令 |
| **状态检查** | <1s | `/status` 命令 |
| **设置更新** | <100ms | 按钮即时响应 |
| **并发用户** | 10+ | 已同时测试 |

### 缓存命中率（预期）

- **完全相同的重复查询**：100% 命中率
- **热门问题**：60–80% 命中率
- **独特查询**：0% 命中率（第一次）

### 资源占用

- **内存**：每个活跃用户会话约 50MB
- **CPU**：很低（空闲 <5%，生成时会尖峰）
- **网络**：每条消息约 10KB（不含 LLM 生成）

## 生产部署

### Webhook 模式（生产推荐）

```bash
# .env for production
TELEGRAM__USE_WEBHOOK=true
TELEGRAM__WEBHOOK_URL=https://your-domain.com
TELEGRAM__WEBHOOK_PATH=/telegram/webhook

# Requires:
# - HTTPS domain with valid certificate
# - Nginx/Caddy for TLS termination
# - Public IP or reverse proxy
```

### 安全最佳实践

1. **限制用户**：私有 bot 使用 `ALLOWED_USER_IDS`
2. **限流**：保持默认限制（20 条/分钟）
3. **环境变量**：永远不要把带真实 token 的 `.env` 提交进仓库
4. **仅 HTTPS**：生产使用 webhook 模式
5. **监控**：通过 Langfuse dashboard 追踪用量

### 扩展考量

高流量部署时：

1. **用户存储**：从内存迁到 Redis/PostgreSQL
2. **缓存**：增加 Redis 内存配额
3. **负载均衡**：多个 API 实例（webhook 模式）
4. **限流**：按你的需求调整
5. **监控**：为错误和延迟设置告警

## 下一步

### 可选增强（第 7.1 周及以后）

- **📸 图片支持**：上传论文 PDF，获得摘要
- **🗣️ 语音消息**：用语音提问（语音转文字）
- **📊 用户分析**：用量模式 dashboard
- **🤝 群聊**：多用户讨论
- **🌍 多语言**：国际化支持
- **🔔 通知**：按类别推送新论文
- **📈 个性化**：基于 ML 的论文推荐
- **🔗 语义缓存**：相似查询的模糊匹配

### 集成想法

- **Slack Bot**：用同一架构移植到 Slack
- **Discord Bot**：扩展到 Discord 社区
- **WhatsApp Bot**：使用 WhatsApp Business API
- **Web Widget**：嵌入网站
- **API Access**：暴露 RESTful API 供集成

## 资源

- **Telegram Bot API**：https://core.telegram.org/bots/api
- **python-telegram-bot**：https://python-telegram-bot.org
- **@BotFather**：https://t.me/botfather
- **Langfuse 文档**：https://langfuse.com/docs
- **Redis 文档**：https://redis.io/docs

## 代码结构

```python
# Entry point: src/main.py
telegram_service = make_telegram_service(...)
await telegram_service.start()

# Service: src/services/telegram/client.py
class TelegramService:
    async def start() -> Start bot in polling/webhook mode
    async def stop() → Stop bot gracefully
    async def health_check() -> Check bot status

# Handlers: src/services/telegram/handlers.py
class TelegramHandlers:
    async def start_command() -> /start
    async def help_command() -> /help
    async def ask_command() -> /ask
    async def search_command() -> /search
    async def settings_command() -> /settings
    async def handle_message() -> Regular text messages

# Formatters: src/services/telegram/formatters.py
format_rag_response() -> Rich Markdown formatting
format_search_results() -> Search result display
format_welcome_message() -> /start message
escape_markdown_v2() -> Telegram MarkdownV2 escaping
split_long_message() -> Auto-split >4000 chars
```

## 成功标准

第 7 周完成当：

- ✅ Bot 响应全部命令
- ✅ 自然语言提问返回 RAG 答案
- ✅ 缓存提供 150–400 倍加速
- ✅ 设置跨会话持久化
- ✅ 交互式键盘可用
- ✅ Langfuse 显示 Telegram traces
- ✅ 错误处理优雅
- ✅ Markdown 格式正确渲染
- ✅ 长消息自动拆分
- ✅ 多用户可并发使用

---

**第 7 周把你的 RAG 系统变成移动优先、可在任意地方通过 Telegram 访问的对话式研究助手！** 🚀

---

## FAQ

**Q: Telegram bot 需要公网 IP 吗？**
A: 不需要！Polling 模式对开发与低流量部署完全够用。Webhook 需要 HTTPS。

**Q: 多个用户能同时用这个 bot 吗？**
A: 可以！每个用户有独立的设置与会话。

**Q: 运行这个 bot 要花多少钱？**
A: 免费！Telegram bot API 完全免费且没有限制。

**Q: 能把 bot 限制给特定用户吗？**
A: 可以！设置 `TELEGRAM__ALLOWED_USER_IDS=123456,789012`，用逗号分隔 Telegram user ID。

**Q: 怎么获取我的 Telegram user ID？**
A: 在 Telegram 上给 `@userinfobot` 发消息，或发一条消息后查看 bot 日志。

**Q: Bot 会存储对话历史吗？**
A: 会，每个用户最近 10 条消息存在内存中。生产环境请迁到 Redis/PostgreSQL。

**Q: 能在没有 Docker 的服务器上部署吗？**
A: 可以！用 `uv sync` 安装依赖后运行 `python src/main.py` 即可。

**Q: 怎么更新 bot token？**
A: 在 `.env` 中更新 `TELEGRAM__BOT_TOKEN` 并重启：`docker compose restart api`

**Q: 能自定义 bot 的人设吗？**
A: 可以！编辑 `src/services/ollama/prompts/` 中的 prompt，以及 `src/services/telegram/formatters.py` 中的消息模板。

**Q: 能和其他 LLM 模型一起用吗？**
A: 可以！改 `OLLAMA_MODEL`，或用 `/settings` 命令选择不同的 Ollama 模型。

---

享受你的新对话式 RAG 界面！🎉
