# 第一阶段 第2周：RAG进阶 + LangChain + 工具调用

> 本周目标：深入理解RAG优化，掌握LangChain框架，学会工具调用
> 
> 上周你已经写出了一个简单RAG，本周让它更好用，学会用框架提升开发效率

| 天 | 上午（2h） | 下午（2h） | 完成打卡 |
|----|------------|------------|----------|
| 第1天 | Embedding模型选型深度对比 | 动手测试不同Embedding效果 | ☐ |
| 第2天 | RAG高级优化技巧原理 | 在项目中实践至少一个优化 | ☐ |
| 第3天 | LangChain核心概念与组件 | 安装跑通官方示例 | ☐ |
| 第4天 | LangChain实现完整RAG | 用LangChain重写上周项目 | ☐ |
| 第5天 | 工具调用：Function Calling / MCP | 实现第一个工具调用示例 | ☐ |
| 第6天 | 两周学习复盘整理 | 修复bug，整理项目到GitHub | ☐ |

---

## 🔢 第1天：Embedding模型选型深度对比

### 上午（2h）：理论对比

### 为什么Embedding这么重要？

> RAG的准确率，**70%取决于Embedding质量**。检索错了，LLM肯定回答错。

### 主流Embedding模型对比

| 模型 | 类型 | 价格 | 部署 | 中文效果 | 推荐场景 |
|------|------|------|------|----------|----------|
| OpenAI text-embedding-3-small | 闭源 | $0.02/1M tokens | 云端 | 不错 | 快速开发，不介意花钱 |
| OpenAI text-embedding-3-large | 闭源 | $0.13/1M tokens | 云端 | 好 | 追求效果，可接受成本 |
| BGE-base-v1.5 | 开源免费 | 免费 | 可本地 | **优秀**（专门优化中文） | 本地部署，对中文要求高 |
| BGE-large-v1.5 | 开源免费 | 免费 | 可本地 | **更好** | 资源足够，追求效果 |
| JinaEmbedding v2 | 开源免费 | 免费 | 可本地 | 好 | 长文本处理 |
| 智联文思Embedding | 国内闭源 | 有免费额度 | 云端 | 优秀 | 国内开发 |

### 新手推荐选择：

- **快速入门**：先用OpenAI Embedding（或通义千问Embedding），不用本地部署，方便
- **中文优先+免费**：用BGE-base，效果比OpenAI 3-small在中文上更好，完全免费

### 什么是"对中文优化更好"？
- 中文分词和英文不同，很多开源Embedding是英文训练的，中文表现差
- BGE是智源研究院做的，专门训练了中文数据，对中文理解更好

---

### 下午（2h）：动手测试

**练习目标：** 用同一个问题和文档，测试两种不同Embedding，对比检索结果

**BGE本地使用步骤：**

1. 安装依赖：
```bash
pip install FlagEmbedding sentence-transformers
```

2. 最小使用示例：
```python
from FlagEmbedding import FlagModel

model = FlagModel('BAAI/bge-base-zh-v1.5')
sentences = [
    "AI Agent是能自主完成复杂任务的智能体", 
    "RAG解决LLM幻觉问题"
]
embeddings = model.encode(sentences)
print(embeddings.shape)  # (2, 768)
```

3. 对比测试：
   - 用上周同一个PDF
   - 分别用两种Embedding生成向量存储
   - 用5个不同问题测试，看哪个返回的结果更相关
   - 把对比结果记在项目README里

**项目地址：** https://github.com/FlagOpen/FlagEmbedding

---

## ⚡ 第2天：RAG优化技巧

### 上午（2h）：学习优化方法

### 1. 分块策略优化

**默认方案：固定大小滑动窗口**
```
chunk_size = 500  # 字符数
overlap = 50      # 重叠，避免把连贯内容切碎
```
优点：简单，够用；缺点：可能把连贯语义切碎。

**进阶方案：语义分块**
- 用Embedding相似度判断，语义连贯的放一块，自动分割
- 工具：LangChain `SemanticChunker`
- 适用：长文档，段落结构清晰

**最佳实践：**
- 短文档：512-1000 token
- 长文档：1000-2000 token
- 一定要留50-100 token重叠

### 2. 混合检索优化

**问题：** 纯向量检索有时候漏了关键词匹配

**解决方案：** 向量检索（语义）+ 关键词检索（BM25）结果融合

```
用户问题 → 
  → 向量检索找Top10
  → BM25关键词检索找Top10
  → 融合结果去重 → 去Top5给LLM
```

效果：比纯向量检索准确率提高5-10%，代码增加不多，强烈建议加上。

### 3. 重排序（Rerank）优化

**为什么需要重排序？**
- 第一步多路召回找Top20
- 用一个更精准（但更慢）的模型对Top20排序，选出最相关的Top5
- 效果：准确率再提升10-15%

**常用重排序模型：**
- CrossEncoder（BERT-based）：效果好，速度相对慢
- BGE-Reranker：中文优化，开源免费，推荐使用

**大致流程：**
```python
from FlagEmbedding import FlagReranker

reranker = FlagReranker('BAAI/bge-reranker-base')

# pairs: (query, passage)
scores = reranker.compute_score([(question, doc1), (question, doc2)])
# 根据score排序，取topN
```

---

### 下午（2h）：动手实践

**任务：** 在你上周的RAG项目中，实践至少一个优化技巧

推荐选择顺序：
1.  **容易见效**：换成BGE中文Embedding → 中文效果立竿见影
2.  **再加一点**：加上重排序 → 准确率进一步提升
3.  **进阶挑战**：实现混合检索

**验收标准：**
- [x] 优化代码已集成到项目
- [x] 在README中记录了优化前后对比
- [x] 能跑通，能看到效果

---

## 🔗 第3天：LangChain v1.x核心概念

### 上午（2h）：框架理解

### 为什么要用LangChain？

你上周手写了RAG，为什么还要用框架？

| 开发阶段 | 手写优势 | 框架优势 |
|----------|----------|----------|
| 学习理解 | 清楚每个步骤，容易理解 | 封装细节，开发快 |
| 功能复杂 | 代码越来越乱，难维护 | 组件化，易扩展 |
| 生态集成 | 每个数据库都要自己适配 | 已经适配了几十种向量库/LLM |

**LangChain就是：** 大模型应用开发的"万能积木"，常用组件都给你做好了，拼起来就行。

### 核心概念理解

#### 1. **PromptTemplate** - 提示词模板
```python
# 不用LangChain你可能会这么写
prompt = f"""你是一个助手，请基于上下文回答：{context}
问题：{question}"""

# 用LangChain更优雅
from langchain.prompts import PromptTemplate

prompt = PromptTemplate.from_template("""
你是一个助手，请基于上下文回答：
{context}
问题：{question}
""")

# 格式化
formatted = prompt.format(context=..., question=...)
```

好处：
- 模板和代码分离
- 参数化，容易复用
- 支持Jinja2等高级模板语法

#### 2. **Chain** - 流程串联
Chain就是把"输入 → A处理 → B处理 → C处理 → 输出"串起来：

```python
# 伪代码理解Chain
chain = prompt | llm | output_parser

# 调用一次就走完所有流程
result = chain.invoke({"question": "XXX"})
```

LangChain的写法：
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()
output_parser = StrOutputParser()

chain = prompt | llm | output_parser
result = chain.invoke({"context": context, "question": question})
```

这就是LangChain的表达式语法（LCEL），简洁清晰。

#### 3. **Retriever** - 检索接口统一
Retriever是一个抽象接口：
```python
# 不管你用Chroma还是PGVector还是Elasticsearch
# 调用方式都是一样的：
results = retriever.get_relevant_documents(query)
```

好处：你换向量库不用改调用代码，切换成本低。

---

### 下午（2h）：动手安装跑通

**步骤：**
1. 安装LangChain：
```bash
pip install langchain langchain-openai langchain-chroma
```

2. 跑通官方最小RAG示例：https://python.langchain.com/docs/tutorials/rag

3. 理解每个部分：
   - `TextLoader` 加载文档
   - `RecursiveCharacterTextSplitter` 分块
   - `Chroma` 向量库存储
   - `retriever` 检索
   - `prompt` 模板
   - `llm` 调用
   - `chain` 串联

**遇到问题：**
- 依赖版本冲突 → 建议用虚拟环境（venv/conda）
- 网络问题 → OpenAI需要科学上网，换成通义千问即可

---

## 🚀 第4天：用LangChain重构RAG项目

### 上午（2h）：学习官方最佳实践

阅读官方文档：https://python.langchain.com/docs/modules/data_connection/

理解：
- 不同文档类型（PDF、Word、Markdown）怎么加载
- 不同分块策略怎么选
- 向量存储有哪些选项

---

### 下午（2h）：动手重构

**任务：** 把你上周手写的RAG项目，用LangChain重构一遍

**预期项目结构：**
```
simple-rag-langchain/
├── .env
├── .gitignore
├── requirements.txt
├── README.md
├── data/
│   └── your-document.pdf
└── main.py
```

**主要步骤：**
1. 用 `PyPDFLoader` 加载PDF
2. 用 `RecursiveCharacterTextSplitter` 分块
3. 用 `Chroma` from_documents 建立索引
4. 定义RAG提示词模板
5. 用LCEL组装成RAG chain
6. 测试问答

**完整最小RAG代码（LangChain v0.1+）：**

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
import os
from dotenv import load_dotenv

load_dotenv()

# 1. 加载PDF
loader = PyPDFLoader("data/your-document.pdf")
documents = loader.load()

# 2. 分块
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(documents)

# 3. 建立向量库
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()

# 4. Prompt模板
prompt = ChatPromptTemplate.from_template("""
你是一个助手，请基于以下上下文回答用户问题。
如果找不到答案，就说"我在文档中没有找到相关信息"，不要编造。

上下文：
{context}

问题：{question}
""")

# 5. 组装RAG链
llm = ChatOpen(model="gpt-3.5-turbo", temperature=0)

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 6. 问答测试
question = "XXX"
answer = rag_chain.invoke(question)
print(answer)
```

对比一下：比你手写是不是简洁很多？LangChain帮你做了哪些事？

---

## 🛠️ 第5天：工具调用 - Function Calling & MCP

### 上午（2h）：原理理解

### 什么是工具调用？

- 没有工具调用：LLM只能聊天，不能"做事"
- 有了工具调用：LLM能识别"什么时候需要调用外部工具"，拿到结果再回答你

**典型例子：查天气**
```
用户：今天北京天气怎么样？

LLM：我需要查天气 → 调用get_weather(city="北京") → 
工具返回：25度，晴天 → 
LLM：今天北京晴天，气温25度。
```

这就是完整的工具调用流程。

### Function Calling（OpenAI标准）

OpenAI的Function Calling格式：

1. 你给LLM说明有哪些工具：
```json
{
  "name": "get_weather",
  "description": "查询城市天气",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string",
        "description": "城市名称"
      }
    },
    "required": ["city"]
  }
}
```

2. LLM返回它要调用哪个函数，参数是什么：
```json
{
  "name": "get_weather",
  "arguments": {
    "city": "北京"
  }
}
```

3. 你调用工具得到结果，发回给LLM，LLM生成最终回答。

**要点：**
- 工具描述一定要写清楚，LLM靠description理解什么时候用这个工具
- 参数说明要准确，类型、说明都不能少

### MCP（Model Context Protocol）

Anthropic推出的**统一工具调用协议**：

解决什么问题：
- 每个LLM厂商Function Calling格式不一样，换模型要改大量代码
- MCP定义统一标准，一次开发，适配所有支持MCP的模型

**发展趋势：** MCP会逐步成为行业标准，值得关注。

---

### 下午（2h）：动手实现简单工具调用

用LangChain实现一个天气查询工具调用示例：

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langgraph.prebuilt import ToolNode
from langgraph.graph import StateGraph, MessagesState
import json

# 1. 定义工具
@tool
def get_weather(city: str) -> str:
    """查询指定城市的天气.
    
    Args:
        city: 城市名称
    """
    # 这里模拟，实际应该调用天气API
    weather_data = {
        "北京": "晴天，25度",
        "上海": "多云，28度",
        "广州": "下雨，30度"
    }
    return weather_data.get(city, f"未知城市{city}")

# 2. 绑定工具到LLM
tools = [get_weather]
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
llm_with_tools = llm.bind_tools(tools)

# 3. 测试调用
from langchain_core.messages import HumanMessage
message = HumanMessage(content="北京今天天气怎么样？")
response = llm_with_tools.invoke([message])

# LLM会返回工具调用请求
print(response.tool_calls)

# 你执行工具调用，把结果返回给LLM得到最终回答
# LangGraph可以帮你自动完成这个循环
```

**完成：** 跑通这个示例，理解工具调用的完整循环。

---

## 📋 第6天：两周学习复盘

### 全天（4h）：整理总结

**要做的事：**

1. **代码整理**（2h）
   - 把两周两个RAG项目（手写版 + LangChain版）整理好
   - 每个项目都要有清晰的README，说明怎么安装怎么运行
   - 修复遇到的bug，把踩过的坑记下来

2. **笔记整理**（1h）
   - 整理学习笔记：
     - 学到了哪些核心概念？
     - 遇到了什么问题？怎么解决的？
     - 有什么经验教训？
   - 记录在你的笔记里，后面复习方便

3. **提交GitHub**（1h）
   - 创建两个仓库，推送到GitHub
   - 检查`.gitignore`有没有忽略`.env`（不要把API Key传上去！）
   - 确认能正常clone下来运行

---

## ✅ 本周学习检查清单

学完本周，你应该掌握：

- [x] 能对比不同Embedding模型的优缺点，选择适合的
- [x] 说出至少3种RAG优化方法，理解原理
- [x] 会用LangChain核心组件：PromptTemplate, Chain, Retriever
- [x] 能用LangChain快速搭建一个RAG应用
- [x] 理解工具调用原理，实现简单的Function Calling
- [x] 两个项目都推送到GitHub，有README说明

---

## 📝 本周测试题（带答案）

> 点击问题标题展开看答案

---

<details>
<summary>▶️ 1. RAG检索中，重排序(Rerank)的作用是什么？</summary>
> A. 加快检索速度  
> B. 提高检索准确率，把最相关结果排前面  
> C. 减少存储占用  
> D. 替换向量Embedding  

**答案：B**  
解释：第一步多路召回拿到Top20，用更精准但较慢的模型重排序选出Top5，能提升10-15%准确率。
</details>

---

<details>
<summary>▶️ 2. LangChain中LCEL是什么？</summary>
> A. LangChain Expression Language，用`|`串联组件  
> B. 一种新的编程语言  
> C. LangChain Extra Library 附加库  
> D. 大语言模型评估标准  

**答案：A**  
解释：LCEL是LangChain的表达式语法，让你能用简洁的`prompt | llm | output_parser`方式组装流程。
</details>

---

<details>
<summary>▶️ 3. 中文推荐用哪个Embedding模型开源免费效果好？</summary>
> A. OpenAI text-embedding-3-small  
> B. BGE-base-zh-v1.5  
> C. GPT-4o Embedding  
> D. Word2Vec  

**答案：B**  
解释：BGE是智源研究院训练的，专门对中文做了优化，开源免费效果比OpenAI原生中文还好。
</details>

---

<details>
<summary>▶️ 4. 什么是混合检索？</summary>
> A. 混合多种不同模型  
> B. 同时用向量检索（语义）+ BM25关键词检索，融合结果  
> C. 混合多个索引  
> D. 同时检索多个文档集合  

**答案：B**  
解释：纯向量有时候漏了关键词匹配，混合检索能互补，提高准确率，代码增加不多，收益明显。
</details>

---

<details>
<summary>▶️ 5. MCP协议解决了什么问题？</summary>
> A. 让模型运行更快  
> B. 统一工具调用格式，换模型不用改代码  
> C. 减少token消耗  
> D. 提高LLM推理准确率  

**答案：B**  
解释：之前每个厂商Function Calling格式都不一样，MCP统一标准，一次开发多处运行。
</details>

---

<details>
<summary>▶️ 6. 代码题：补全LangChain RAG代码</summary>

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. 加载PDF
loader = ____①__("data/doc.pdf")
documents = loader.load()

# 2. 分块
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.____②____(documents)

# 3. 建立向量库
vectorstore = Chroma.____③____(
    documents=splits,
    embedding=OpenAIEmbeddings()
)
retriever = vectorstore.____④____()

# 运行问答
question = "什么是RAG"
answer = rag_chain.invoke(question)
print(answer)
```

**答案：**
① `PyPDFLoader`  
② `split_documents`  
③ `from_documents`  
④ `as_retriever`

**完整可运行代码就是教程中给出的，可以直接运行。**
</details>

---

## 📚 扩展阅读（选看）

- [LangChain官方文档](https://python.langchain.com/docs/)
- [BGE官方GitHub](https://github.com/FlagOpen/FlagEmbedding)
- [MCP模型上下文协议](https://modelcontextprotocol.io/)
- [LangGraph官方文档](https://langchain-ai.github.io/langgraph/) - LangChain的Agent编排框架
