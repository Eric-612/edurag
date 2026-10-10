# EduRAG 学科智能答疑系统

面向 IT 职业教育的智能答疑系统，基于 RAG 架构解决大模型幻觉与私有知识更新问题。

## 项目介绍

EduRAG 融合 FAQ 库与多学科（AI/Java/测试/运维/大数据）课程文档，支持多轮对话、学科筛选与流式作答，替代人工客服自动答疑。

### 核心特性

- **两级链路**：FAQ 短答 + RAG 兜底，降低大模型成本并抑制幻觉
- **混合检索**：Milvus 稠密+稀疏向量 RRF 融合召回，bge-reranker 重排
- **父子分块**：兼顾检索精度与上下文完整性
- **查询分类**：BERT 微调查询意图，动态选择最优检索策略
- **流式输出**：WebSocket 逐 token 流式返回

## 技术栈

| 类别 | 技术 |
|------|------|
| 后端框架 | Python / FastAPI |
| RAG 框架 | LangChain |
| 向量数据库 | Milvus |
| 嵌入模型 | bge-m3 |
| 重排模型 | bge-reranker-large |
| 关键词检索 | BM25 + jieba |
| 大模型 | Qwen-Max / DashScope |
| 关系型数据库 | MySQL |
| 缓存 | Redis |
| 通信协议 | WebSocket / SSE |

## 项目结构

```
integrated_qa_system/
├── base/                   # 基础配置
│   └── config.py            # 配置加载
├── mysql_qa/               # FAQ 短答模块
│   ├── db/                 # MySQL 操作
│   ├── cache/              # Redis 缓存
│   └── retrieval/         # BM25 检索
├── rag_qa/                 # RAG 检索模块
│   ├── core/               # RAG 核心逻辑
│   │   ├── rag_system.py   # RAG 系统主类
│   │   ├── vector_store.py # 向量存储
│   │   ├── query_classifier.py # 查询分类
│   │   └── strategy_selector.py # 策略选择
│   ├── data/               # 课程文档数据
│   ├── edu_document_loaders/ # 文档加载器
│   ├── edu_text_spliter/   # 文本分割器
│   ├── models/             # 本地模型（gitignore）
│   └── rag_assesment/      # RAG 评估
├── static/                 # 前端页面
├── logs/                   # 日志（gitignore）
├── app.py                  # FastAPI 主入口
├── api.py                  # API 接口定义
├── new_main.py             # 主程序
├── config.ini.example      # 配置示例
└── requirments.txt         # 依赖列表
```

## 快速开始

### 环境要求

- Python 3.10+
- MySQL 8.0+
- Redis 6.0+
- Milvus 2.0+

### 安装步骤

1. **克隆仓库**
```bash
git clone https://gitee.com/eric050612/edurag.git
cd edurag
```

2. **安装依赖**
```bash
pip install -r requirments.txt
```

3. **配置环境**
```bash
cp config.ini.example config.ini
# 编辑 config.ini，填入你的数据库密码、API Key 等
```

4. **配置环境变量**
```bash
export ALIYUN_API_KEY=your_dashscope_api_key
```

5. **下载模型**
```bash
# 模型较大，请手动下载放到 rag_qa/models/ 目录
# bge-m3、bge-reranker-large、bert-base-chinese
```

6. **启动服务**
```bash
python app.py
```

## 核心设计

### 两级问答链路

```
用户问题
   ↓
BM25 匹配 FAQ 库
   ├─ 相似度 ≥ 0.85 → 直答 + Redis 缓存
   └─ 未命中 → 走 RAG 链路
              ↓
         查询分类（BERT）
              ├─ 通用问题 → 直接检索
              └─ 专业问题 → LLM 选择策略
                          ├─ 直接检索
                          ├─ HyDE
                          ├─ 子查询分解
                          └─ 回溯检索
              ↓
         Milvus 混合召回（稠密+稀疏 RRF）
              ↓
         bge-reranker 重排
              ↓
         LLM 生成答案
              ↓
         流式输出
```

### 父子分块策略

- 父块：1200 字（上下文完整）
- 子块：300 字（检索精度）
- 重叠：50 字
- 子块向量检索，返回父块原文

## 许可证

MIT License
