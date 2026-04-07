# 第一阶段 第1周：Agent基础认知 + API调用 + RAG核心

> 本周目标：建立Agent基础认知，完成第一个可运行项目——私人知识库问答机器人
> 
> **学习路径：** 从概念理解 → API调用 → 核心技术(RAG) → 完整项目，循序渐进

| 天 | 上午（2h） | 下午（2h） | 完成打卡 |
|----|------------|------------|----------|
| 第1天 | Agent核心概念入门，四大组件深度理解 | 热门Agent案例分析，亲身体验现有平台 | ☐ |
| 第2天 | Prompt工程核心技巧：角色扮演、思维链、少样本 | 结构化Prompt + 多轮对话管理实践 | ☐ |
| 第3天 | LLM API调用原理，请求/响应格式理解 | 实现流式输出，封装命令行聊天机器人 | ☐ |
| 第4天 | RAG原理精讲：为什么能解决LLM幻觉问题 | RAG完整流程拆解，每个步骤动手理解 | ☐ |
| 第5天 | 向量数据库基础，Chroma从零上手 | 完整RAG流程代码实践 | ☐ |
| 第6天 | 本周项目：私人知识库问答机器人 | 代码优化 + README编写 + GitHub提交 | ☐ |

---

## 📖 第1天：Agent核心概念入门

### 上午（2h）：理论理解

#### 1. 什么是AI智能体（Agent）？

**通俗理解：**
- 普通ChatGPT = "你问一句，它答一句"，一问一答
- **AI Agent** = 给LLM装上手和脚，能**自主规划**、**调用工具**、**记忆信息**、**一步步完成复杂任务**

**类比理解：**
- 普通LLM：一个只会出主意的顾问
- AI Agent：一个能自己写方案、查资料、调用工具、最终交付结果的项目经理

#### 2. Agent四大核心组件（必须理解）

| 组件 | 作用 | 类比 |
|------|------|------|
| **大脑（LLM）** | 理解用户需求，做决策，生成回答 | 公司CEO |
| **规划模块** | 把大目标拆成小步骤，按顺序执行 | 项目拆解 |
| **工具调用** | 联网搜索、查数据库、调用API、执行代码 | 员工干活 |
| **记忆模块** | 存储历史对话和中间结果，长期上下文 | 笔记本 |

**深入阅读材料：**
- 📺 视频：李宏毅《AI Agent》第一讲 → B站直接搜索，通俗易懂
- 📄 文章：[AI智能体(Agent)保姆级入门指南](https://zhuanlan.zhihu.com/p/199509730184530938)

**思考题（花10分钟想想）：**
- 生活中哪些问题适合用Agent解决？
- 哪些问题不适合？为什么？

---

### 下午（2h）：案例体验

#### 1. 看5个代表性Agent案例

| Agent | 特点 | 学习点 |
|-------|------|--------|
| **Devin** | AI软件工程师，能独立写代码、改bug、部署 | Agent能完成复杂软件工程任务 |
| **AutoGPT** | 最早的开源自主Agent，探索Agent可能性 | 自主规划+工具调用的原型实现 |
| **GPTs** | OpenAI的自定义GPT，支持自定义功能 | 低代码构建Agent的商业产品 |
| **Coze/扣子** | 国内Agent开发平台，拖拽式搭建 | 快速体验不需要写代码 |
| **Klarna客服Agent** | 商业落地，处理用户咨询和下单 | Agent在客服场景的规模化应用 |

#### 2. 动手实践：去Coze体验现成Agent

1.  打开 https://www.coze.cn/ 注册登录
2.  发现广场找几个热门Agent（比如AI简历顾问、Python编程助手）
3.  实际用一下，感受Agent和普通ChatGPT有什么不同

**记录：** 你觉得Agent哪里好用？哪里还不够好？

---

## 📝 第2天：Prompt工程基础

### 上午（2h）：核心技巧学习

**为什么要学Prompt工程？**
> 再好的Agent框架，Prompt不对也出不了好结果。Prompt是你和LLM对话的语言。

#### 三大核心技巧必须掌握：

##### 1️⃣ 角色扮演（Role Playing）

**原理：** 让LLM明确身份，输出更符合预期

**不好的Prompt：**
```
给我写一个求职简历
```

**好的Prompt：**
```
你是一个有10年经验的互联网HR，精通技术岗位简历优化。
现在帮我写一份Python后端开发工程师的求职简历，
要求突出项目经验和技术栈，符合HR看简历的习惯。
```

**动手练习：** 让LLM扮演你的行业专家，对比两种方式输出差异。

##### 2️⃣ 思维链（Chain-of-Thought, CoT）

**原理：** 让LLM一步步思考，而不是直接给答案，正确率提升30%+

**Prompt模板：**
```
问题：XXX
让我们一步一步思考来解决这个问题：
```

**动手练习：** 找一道数学题或逻辑题，测试加不加思维链的差异。

##### 3️⃣ 少样本学习（Few-Shot Learning）

**原理：** 给几个例子，LLM就能学会你要的格式，比说一堆描述管用

**示例：**
```
我要你把用户问题分类到以下类别之一：技术问题、产品问题、售后问题。

示例1：
用户：这个软件闪退怎么办？
分类：技术问题

示例2：
用户：你们新款什么时候发售？
分类：产品问题

现在分类这个问题：用户退款怎么申请？
分类：
```

**学习资源：**
- 📺 视频：【2小时彻底掌握提示词工程】https://www.bilibili.com/video/BV1kZ421s7km/

---

### 下午（2h）：结构化Prompt实战

#### 1. 使用LangChain PromptTemplate

当你需要重复使用同一个Prompt模板时，用`PromptTemplate`来管理：

```python
from langchain.prompts import PromptTemplate

# 定义模板
prompt = PromptTemplate.from_template(
    "你是一个{role}专家，请回答用户的问题：{question}"
)

# 格式化使用
formatted = prompt.format(role="AI Agent开发", question="什么是工具调用？")
print(formatted)
```

**输出结果：**
```
你是一个AI Agent开发专家，请回答用户的问题：什么是工具调用？
```

好处：
- 代码整洁，便于维护
- 参数化，灵活复用
- 统一管理Prompt

#### 2. 让LLM输出稳定JSON格式

在开发中，经常需要LLM输出JSON供程序解析，关键是**在Prompt中说明格式要求**：

```python
prompt = """
请分析用户查询的意图，输出JSON格式：
{{
  "intent": "查询天气/导航/聊天",
  "confidence": 0.95,
  "parameters": {{
    "city": "北京"
  }}
}}

用户查询：{query}
"""
```

**常见坑：**
- ❌ 不要只说"输出JSON"，要**给具体格式示例**
- ✅ 最好用JSON Schema说明每个字段类型

#### 3. 多轮对话历史管理

LLM是无状态的，每次调用都要把历史对话发过去：

```python
# 对话历史存在列表里
messages = [
    {"role": "system", "content": "你是一个助手"},
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好，有什么可以帮你？"},
    # ... 继续添加新对话
]

# 每次调用都把完整history发给LLM
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=messages
)
```

**优化技巧：** 上下文窗口有限，只保留最近N轮：
```python
# 只保留最近10轮，节省token
if len(messages) > 10:
    messages = messages[-10:]
```

---

## 🔌 第3天：LLM API调用实战

### 上午（2h）：基础调用

#### 1. 选择一个大模型API

对于入门，推荐：

| 平台 | 国内访问 | 免费额度 | 推荐指数 |
|------|----------|----------|----------|
| OpenAI | 需要科学上网 | 有计费门槛 | ⭐⭐⭐⭐⭐ |
| 通义千问 | 国内直接用 | 有免费额度 | ⭐⭐⭐⭐⭐ |
| 豆包 | 国内直接用 | 有免费额度 | ⭐⭐⭐⭐ |
| Anthropic | 需要科学上网 | 计费 | ⭐⭐⭐⭐ |

#### 2. 获取API Key

**通用步骤：**
1.  注册账号
2.  进入个人设置 → API Keys
3.  创建新的API Key
4.  **保存好Key，只显示一次！**

> ⚠️ **安全提醒：** API Key是要钱的，绝对不要提交到GitHub公开仓库！
> 用环境变量读取，不要硬编码在代码里。

#### 3. 第一个API调用

以OpenAI为例，最小可运行代码：

```python
import openai
import os

# 从环境变量读取API Key，不要硬编码！
openai.api_key = os.getenv("OPENAI_API_KEY")

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你是一个助手"},
        {"role": "user", "content": "你好！介绍一下你自己"},
    ]
)

# 打印回答
print(response.choices[0].message.content)
```

**通义千问示例（阿里云）：**
```python
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen-plus",
    messages=[
        {"role": "system", "content": "你是一个助手"},
        {"role": "user", "content": "你好！"}
    ]
)
print(response.choices[0].message.content)
```

**常见问题：**
- Q: 连接超时 → A: 检查网络/防火墙，OpenAI需要科学上网
- Q: 认证错误 → A: 检查API Key是否正确，是否有余额

---

### 下午（2h）：流式输出封装

#### 1. 什么是流式输出？

- 非流式：等LLM全部生成完才返回，用户要等好几秒
- 流式：生成一个token返回一个，一点点显示出来，体验好

**OpenAI流式调用示例：**

```python
import openai

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[...],
    stream=True  # 开启流式
)

for chunk in response:
    if chunk.choices[0].delta.get("content"):
        # 逐块输出，实现打字机效果
        print(chunk.choices[0].delta.content, end="", flush=True)
```

#### 2. 项目：封装命令行聊天机器人

最终目标：一个支持多轮对话、流式输出的命令行机器人，完整代码大约50行：

**功能要求：**
- [ ] 支持多轮对话，记住上下文
- [ ] 流式输出，逐字显示
- [ ] 输入`exit`退出
- [ ] 用环境变量读API Key，不硬编码

**参考架构：**
```
simple_chat/
├── .env                # 存API Key（加入.gitignore！）
├── .gitignore
├── README.md
└── chat.py             # 主程序，约50行
```

---

## 🔍 第4天：RAG检索增强生成原理

### 上午（2h）：原理解析

#### 为什么需要RAG？LLM有三个痛点：

1.  **幻觉问题**：一本正经胡说八道，事实性错误
2.  **知识截止**：训练数据之后的新知识不知道
3.  **私有数据**：你的公司文档、个人笔记，LLM没见过

**RAG怎么解决？**

> 给LLM开卷考试！回答问题前先从你的知识库找相关信息，把找到的信息拼到Prompt里给LLM，让LLM基于准确信息回答。

```
用户问题 → 从向量数据库检索相关文档 → 把问题+相关文档一起给LLM → LLM回答
```

#### 对比：RAG vs 微调

| 对比项 | RAG | 微调（Fine-tuning） |
|--------|-----|---------------------|
| 知识更新 | 容易，换知识库就行 | 困难，要重新训练 |
| 成本 | 低，只用推理不训练 | 高，需要GPU训练 |
| 幻觉减少 | 有效，来源可追溯 | 不一定，还是可能幻觉 |
| 适合场景 | 知识库问答、文档检索 | 风格对齐、特定任务 |

**结论：** Agent开发90%场景先上RAG，效果不够再考虑微调。

---

### 下午（2h）：RAG流程拆解

完整RAG六大步骤，每个步骤做什么：

| 步骤 | 作用 | 技术要点 |
|------|------|----------|
| **1. 文档加载** | 把不同格式文件转成文本 | PDF: PyPDF, Word: python-docx, 网页: requests+BeautifulSoup |
| **2. 文本分块** | 把长文档切成小段，太长塞不下上下文 | 一般切500-1000token，有重叠更好 |
| **3. 向量化** | 把每个文本块转成Embedding向量 | 用OpenAI Embedding或开源模型如BGE |
| **4. 存储** | 把向量和原文存在向量数据库 | 方便按相似性快速检索 |
| **5. 检索** | 用户问题转向量，找最相似的Topk个块 | 一般取Top3-Top5，拼进Prompt |
| **6. 生成回答** | LLM基于检索到的上下文回答 | Prompt模板："基于以下上下文回答问题，如果不知道就说不知道..." |

**图解：**
```
[大文档] → [切成小块] → [转成向量] → [存在向量库]
                              ↑
[用户问题] → [转成向量] → [找相似向量] → [取出原文块] → [给LLM] → [回答]
```

**思考题：**
- 为什么分块不能太大也不能太小？
- 为什么要找Top3-Top5，不找只找Top1？

**学习资源：**
- 📄 文章：[RAG 入门介绍](https://www.zhihu.com/question/592135908/answer/2987353945)

---

## 🗄️ 第5天：向量数据库实战

### 上午（2h）：向量基础 + Chroma安装

#### 什么是向量Embedding？

- 把一段文字转换成一串数字（比如1024维向量）
- **语义相似的文字，向量距离近**
- 所以可以通过找最近的向量，找到语义相似的文本

示例：
```
"机器学习" → [0.12, -0.34, 0.56, ...]
"AI" → [0.15, -0.31, 0.52, ...]  (距离近，相似)
"红烧肉" → [0.98, 0.76, -0.12, ...]  (距离远，不相似)
```

#### 入门推荐：Chroma向量数据库

**为什么选Chroma：**
- ✅ 纯Python，pip一键安装
- ✅ 轻量级，适合开发学习
- ✅ 不需要额外启动服务，本地文件存储
- ✅ API简单，几分钟就能跑通

**安装：**
```bash
pip install chromadb
```

**最小示例：**

```python
import chromadb

# 客户端初始化
client = chromadb.Client()

# 创建集合
collection = client.create_collection("my_docs")

# 添加文档
collection.add(
    documents=[
        "AI Agent是能自主完成复杂任务的智能体",
        "RAG是检索增强生成，用于解决LLM幻觉问题"
    ],
    ids=["doc1", "doc2"]
)

# 检索
results = collection.query(
    query_texts=["什么是Agent"],
    n_results=2
)

print(results)
```

跟着官方文档走一遍：https://docs.trychroma.com/getting-started

---

### 下午（2h）：完整RAG实践

目标：把你的一个PDF文档导入Chroma，实现问答检索

完整代码框架：

```python
import PyPDF2
import chromadb

# 1. 读取PDF
def read_pdf(path):
    text = ""
    with open(path, "rb") as f:
        reader = PyPDF2.PdfReader(f)
        for page in reader.pages:
            text += page.extract_text() + "\n"
    return text

# 2. 分块
def split_text(text, chunk_size=500, overlap=50):
    # 简单按字符分块，入门够用了
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks

# 3. 存入Chroma
client = chromadb.Client()
collection = client.create_collection("my_knowledge")
# ... 继续完成

# 4. 测试检索
question = "XXX"
results = collection.query(query_texts=[question], n_results=3)
for doc in results['documents'][0]:
    print("--- 检索结果 ---")
    print(doc)
```

**练习步骤：**
1. 找一个你自己的PDF（论文、笔记、电子书都可以）
2. 运行代码，导入到Chroma
3. 输入几个相关问题，看检索结果对不对
4. 观察：返回的段落是不是真的和问题相关？

---

## 🚀 第6天：本周项目 - 私人知识库问答机器人

### 全天（4h）：整合所有内容，做一个完整项目

**项目功能：**
- 用户上传PDF → 自动分割、向量化、存储
- 用户问问题 → 检索相关段落 → LLM基于段落回答
- 支持引用来源，用户可以验证回答是否正确

**项目结构：**
```
simple-rag/
├── .env                    # API Key（加入.gitignore）
├── .gitignore
├── requirements.txt        # 依赖列表
├── README.md               # 项目说明
├── pdfs/                   # 存放你的PDF文件
└── main.py                 # 主程序
```

**核心Prompt模板：**
```
你是一个助手，请基于提供的上下文回答用户问题。
如果上下文中没有找到答案，就直接说"我在文档中没找到相关信息"，不要编造。

上下文：
{context}

用户问题：{question}

回答：
```

**完成标准：**
- [ ] 能正常跑通，端到端流程工作
- [ ] 回答准确，不瞎编
- [ ] 有README说明怎么使用
- [ ] 推送到GitHub

**参考项目：** https://github.com/microsoft/ai-agents-for-beginners/blob/main/03-rag/README.md

---

## ✅ 本周学习检查清单

学完本周，你应该掌握：

- [ ] 能说清楚Agent是什么，四大组件各自作用
- [ ] 掌握三大Prompt技巧：角色扮演、思维链、少样本
- [ ] 会调用LLM API，实现流式输出和多轮对话
- [ ] 理解RAG原理和完整流程
- [ ] 会用Chroma做向量检索
- [ ] 独立完成RAG项目，代码可运行

---

---

## 📝 本周测试题（带答案）

> 点击「▶️」展开看答案

---

<details>
<summary>▶️ 1. 以下哪个不是Agent四大核心组件？</summary>
> A. 大脑（LLM）  
> B. 规划模块  
> C. 数据库  
> D. 记忆模块  

**答案：C**  
解释：Agent四大组件是：大脑(LLM)、规划、工具调用、记忆。数据库是存储工具，不是核心组件。
</details>

---

<details>
<summary>▶️ 2. 思维链（CoT）的核心作用是什么？</summary>
> A. 让LLM输出更快  
> B. 让LLM一步步思考，提高推理正确率  
> C. 让LLM能调用工具  
> D. 让LLM记住对话历史  

**答案：B**  
解释：思维链就是让LLM把思考过程说出来，正确率能提升30%以上。
</details>

---

<details>
<summary>▶️ 3. RAG主要解决LLM的什么问题？（多选）</summary>
> A. 幻觉问题  
> B. 速度慢问题  
> C. 知识截止问题  
> D. 私有数据问题  

**答案：A、C、D**  
解释：RAG不能解决速度慢问题，反而因为多了检索步骤会稍慢一点，但能有效解决幻觉、知识截止、私有数据这三个问题。
</details>

---

<details>
<summary>▶️ 4. 以下关于RAG分块的说法，哪个正确？</summary>
> A. 分块越大越好，信息完整  
> B. 分块越小越好，检索精确  
> C. 一般分500-1000token比较合适，要留重叠  
> D. 不用分块，直接整个文档存进去  

**答案：C**  
解释：太大占上下文，检索不精确；太小切碎语义；一般500-1000token最合适，留一点重叠避免切碎连贯内容。
</details>

---

<details>
<summary>▶️ 5. 代码题：补全下面的OpenAI API调用代码</summary>

```python
import openai
import os

openai.api_key = ____①____

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": ____②____, "content": "你是一个助手"},
        {"role": ____③____, "content": "你好"},
    ]
)

print(response.____④____)
```

**答案：**
① `os.getenv("OPENAI_API_KEY")`  
② `"system"`  
③ `"user"`  
④ `choices[0].message.content`

**完整可运行代码：**
```python
import openai
import os

openai.api_key = os.getenv("OPENAI_API_KEY")

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你是一个助手"},
        {"role": "user", "content": "你好"},
    ]
)

print(response.choices[0].message.content)
```
</details>

---

<details>
<summary>▶️ 6. 代码题：Chroma检索，补全代码</summary>

```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("my_docs")

collection.____①____(
    documents=["Python是一种编程语言", "AI Agent能自主完成任务"],
    ids=["doc1", "doc2"]
)

results = collection.____②____(
    query_texts=["什么是AI Agent"],
    n_results=2
)

print(results)
```

**答案：**
① `add`  
② `query`

**运行结果：** 会返回第二个文档"AI Agent能自主完成任务"作为最相关结果。
</details>

---

## 📚 扩展阅读（选看）

- [LangChain官方文档 - RAG](https://python.langchain.com/docs/modules/data_connection/)
- [Chroma官方文档](https://docs.trychroma.com/)
- [LlamaIndex文档](https://docs.llamaindex.ai/) - 另一个流行的RAG框架
