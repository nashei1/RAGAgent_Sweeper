# 智扫通 - 扫地机器人智能客服系统

基于 RAG（检索增强生成）和 ReAct Agent 的智能客服聊天机器人，为扫地机器人和扫拖一体机器人用户提供专业的问答服务和个性化使用报告。

## 功能特性

### RAG 智能问答
基于知识库的检索增强生成，能够准确回答扫地机器人使用、故障排除、维护保养、选购指南等问题。

### ReAct Agent 推理
采用"思考 → 行动 → 观察"的推理循环，Agent 能够自主决策调用合适的工具完成复杂任务。

### 个性化使用报告
根据用户使用数据生成包含清洁效率、耗材状态、维护建议的个性化 Markdown 报告。

### 动态提示词切换
通过中间件机制实现上下文感知的系统提示词动态切换，支持对话模式和报告生成模式无缝转换。

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Streamlit |
| LLM 框架 | LangChain / LangGraph |
| 对话模型 | 通义千问 `qwen3-max` |
| 向量模型 | DashScope `text-embedding-v4` |
| 向量数据库 | ChromaDB |
| 文本分割 | RecursiveCharacterTextSplitter |

## 项目结构

```
RAGAgent_Sweeper/
├── app.py                      # Streamlit 应用入口
├── agent/
│   ├── react_agent.py          # ReAct Agent 定义
│   └── tools/
│       ├── agent_tools.py      # 工具定义（RAG、天气、用户信息等）
│       └── middleware.py       # 中间件（监控、日志、提示词切换）
├── config/
│   ├── agent.yml               # 外部数据路径配置
│   ├── chroma.yml              # ChromaDB 向量存储配置
│   ├── prompts.yml             # 提示词文件路径
│   └── rag.yml                 # 模型名称配置
├── data/
│   ├── external/
│   │   └── records.csv         # 模拟用户使用记录
│   ├── 扫地机器人100问.pdf      # 扫地机器人常见问题
│   ├── 扫地机器人100问2.txt     # 扫地机器人常见问题（续）
│   ├── 扫拖一体机器人100问.txt  # 扫拖一体机器人常见问题
│   ├── 故障排除.txt             # 故障排除指南
│   ├── 维护保养.txt             # 维护保养指南
│   └── 选购指南.txt             # 选购指南
├── model/
│   └── factory.py              # 模型工厂（对话模型 + 向量模型）
├── prompts/
│   ├── main_prompt.txt         # 主系统提示词
│   ├── rag_summarize.txt       # RAG 摘要提示词模板
│   └── report_prompt.txt       # 报告生成系统提示词
├── rag/
│   ├── rag_service.py          # RAG 摘要服务
│   └── vector_store.py         # ChromaDB 向量存储服务
└── utils/
    ├── config_handler.py       # YAML 配置加载器
    ├── file_handler.py         # 文件工具（MD5、PDF/TXT 加载）
    ├── logger_handler.py       # 日志配置
    ├── path_tool.py            # 路径工具
    └── prompt_loader.py        # 提示词加载器
```

## 快速开始

### 环境要求

- Python 3.9+
- Conda（推荐）或 venv

### 安装步骤

1. 克隆项目
```bash
git clone <repository-url>
cd RAGAgent_Sweeper
```

2. 创建并激活虚拟环境
```bash
conda create -n ragsweeper python=3.10
conda activate ragsweeper
```

3. 安装依赖
```bash
pip install streamlit langchain langchain-core langchain-community langchain-chroma langchain-text-splitters langgraph pyyaml pypdf
```

4. 配置 API Key

设置阿里云 DashScope API Key：
```bash
export DASHSCOPE_API_KEY="your-api-key-here"
```

### 运行应用

```bash
streamlit run app.py
```

应用将在浏览器中打开，默认地址为 `http://localhost:8501`。

## 配置说明

所有配置文件位于 `config/` 目录：

| 文件 | 说明 |
|------|------|
| `rag.yml` | 模型名称配置（对话模型、向量模型） |
| `chroma.yml` | ChromaDB 配置（集合名称、持久化目录、top-k、分块参数） |
| `prompts.yml` | 提示词文件路径配置 |
| `agent.yml` | 外部数据文件路径配置 |

## 知识库

`data/` 目录包含以下知识文档：

- **扫地机器人100问.pdf** - 扫地机器人常见问题解答
- **扫地机器人100问2.txt** - 扫地机器人常见问题补充
- **扫拖一体机器人100问.txt** - 扫拖一体机器人常见问题
- **故障排除.txt** - 故障诊断与排除指南
- **维护保养.txt** - 日常维护与保养建议
- **选购指南.txt** - 产品选购建议

首次运行时，系统会自动加载这些文档到 ChromaDB 向量数据库中（持久化到 `chroma_db/` 目录），并使用 MD5 去重避免重复索引。

## 架构设计

### Agent 工作流程

```
用户提问
    ↓
ReAct Agent（思考）
    ↓
选择工具（行动）
    ↓
执行工具（观察）
    ↓
生成回答 / 继续推理
```

### 工具列表

| 工具名称 | 功能 |
|----------|------|
| `rag_summarize` | 检索知识库并生成摘要 |
| `get_weather` | 获取天气信息（模拟） |
| `get_user_location` | 获取用户位置（模拟） |
| `get_user_id` | 获取用户 ID（模拟） |
| `get_current_month` | 获取当前月份（模拟） |
| `fetch_external_data` | 加载外部用户数据 |
| `fill_context_for_report` | 触发报告生成模式 |

### 中间件机制

- **monitor_tool** - 工具执行监控与日志记录
- **log_before_model** - 模型调用前的日志记录
- **report_prompt_switch** - 动态切换系统提示词到报告生成模式