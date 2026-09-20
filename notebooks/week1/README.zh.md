# 第 1 周：基础设施搭建与验证

> 本文为 [README.md](./README.md) 的一对一中文译本。

本目录包含 arXiv Paper Curator 项目第 1 周材料，重点是搭建并验证完整基础设施栈。

## 内容

### `week1_setup.ipynb`
一份完整的 Jupyter notebook，引导学生完成：

1. **系统需求与搭建**
   - 理解每个技术组件及其用途
   - 跨平台安装说明（Windows、macOS、Linux）
   - 用自动化检查验证前置条件

2. **基础设施架构**
   - 多服务架构的完整概览
   - 理解 Docker 容器如何通信
   - 数据持久化与 volume 管理概念

<p align="center">
  <img src="../../static/week1_infra_setup.png" alt="第 1 周基础设施搭建" width="700">
</p>

**架构概览：**
- **FastAPI**（端口 8000）：带异步支持与自动文档的 REST API
- **PostgreSQL 16**（端口 5432）：论文元数据与正文的主数据库
- **OpenSearch 2.19**（端口 9200、5601）：带管理 Dashboard 的混合检索引擎
- **Apache Airflow 3.0**（端口 8080）：带 DAG 与 PostgreSQL 后端的工作流编排
- **Ollama**（端口 11434）：供后续 RAG 使用的本地 LLM 服务器
- **Docker Network**：全部服务通过 `rag-network` 通信，并使用持久化 volume

3. **逐个服务搭建**
   - 用于论文元数据存储的 PostgreSQL
   - 用于全文检索的 OpenSearch
   - 用于工作流自动化的 Apache Airflow
   - 用于本地 LLM 推理的 Ollama
   - 用于 REST API 的 FastAPI

4. **验证与测试**
   - 全部服务的自动化健康检查
   - 逐步验证流程
   - 模块化 Ollama 测试（4 个聚焦的测试 cell）
   - 常见故障场景与解决方案

## 学习目标

完成本周材料后，学员将能够：

- 理解容器化与 Docker Compose 编排
- 学会搭建生产级基础设施栈
- 获得数据库设计与 API 开发经验
- 掌握多服务应用的故障排除技巧
- 学习直接 HTTP API 测试 vs 服务抽象层
- 建立使用专业开发工具的信心

## Ollama 测试（第 1 周简化版）

notebook 包含拆成多个聚焦 cell 的模块化 Ollama 测试：

- **Test 3A**：检查可用模型
- **Test 3B**：简单模型测试（若已安装模型）
- **Test 3C**：性能分析
- **Test 3D**：学习笔记与安装命令

### 简易模型安装（第 1 周可选）

```bash
# Using Makefile (recommended)
make ollama-pull MODEL=llama3.2:1b
make ollama-test MODEL=llama3.2:1b

# Direct HTTP calls for learning
curl -X POST http://localhost:11434/api/pull -d '{"name":"llama3.2:1b"}'
curl -X POST http://localhost:11434/api/generate -d '{"model":"llama3.2:1b","prompt":"Hello","stream":false}'
```

### 课程推荐模型

- **llama3.2:1b**（1.2GB）- 快，适合测试
- **llama3.2:3b**（2.0GB）- 速度与质量的平衡
- **llama3.1:8b**（4.7GB）- 质量更好，更慢

**注意**：第 1 周不需要任何模型——服务健康检查在没有模型时也能工作。

## 目标读者

本材料面向：
- 想学习现代软件基础设施的**初学者**
- 想理解真实应用如何构建的**学生**
- 转向软件开发或 DevOps 的**从业者**
- 任何想构建自己的 AI 研究工具的人

## 时间投入

- **搭建**：2–3 小时（含软件安装与下载）
- **完成 notebook**：1 小时
- **合计**：2–4 小时

## 📖 额外资源

**第 1 周博客：** [The Infrastructure That Powers RAG Systems](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag)
- 深入每个基础设施组件
- 生产部署考量
- 架构决策说明

## 支持资源

如果遇到问题：
1. 查看 notebook 中的故障排除章节
2. 回顾常见问题与解决方案
3. 确认所有前置条件已正确安装
4. 按逐步验证流程操作
5. 在 Jam With AI substack 聊天频道提问

## 下一步

完成第 1 周后，你将能够：
- 理解每个服务如何贡献于整体系统
- 按需修改和扩展基础设施
- 进入第 2 周：arXiv 集成与 PDF 处理
- 更有信心地使用专业开发环境
