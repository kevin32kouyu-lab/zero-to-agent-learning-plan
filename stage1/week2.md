# 第一阶段 第2周

> 本周目标：深入RAG，掌握LangChain基础，理解工具调用标准

| 天 | 上午（2h） | 下午（2h） | 完成打卡 |
|----|------------|------------|----------|
| 第1天 | Embedding模型选型，对比开源vs闭源 | 测试不同Embedding在你的RAG上效果 | ☐ |
| 第2天 | RAG优化技巧：分块策略、混合检索、重排序 | 在你的项目中实践一个优化技巧 | ☐ |
| 第3天 | LangChain v1.x核心概念：Chain、PromptTemplate、Retriever | 安装LangChain，跑通官方示例 | ☐ |
| 第4天 | LangChain实现完整RAG流程 | 把你上周的RAG项目用LangChain重写一遍 | ☐ |
| 第5天 | Function Calling vs MCP协议，理解工具调用标准 | 实现一个简单的工具调用示例 | ☐ |
| 第6天 | 项目复盘，整理笔记，提交GitHub | 回顾两周基础学习，总结收获 | ☐ |

---

## 每日详细内容（带具体资源链接）

### 第1天：Embedding模型选型

**上午（2h）**
1. (1h) 理解Embedding作用：把文本转成向量，通过向量相似性找到相关文本，是RAG的核心基础
2. (1h) 对比不同选型：
   - 闭源Embedding：OpenAI `text-embedding-3` 效果好，按token收费，方便
   - 开源Embedding：**BGE系列（中文推荐）**、JinaEmbedding，可本地部署免费，对中文优化更好

**下午（2h）**
1. (2h) **动手测试**：用同一个问题，同一个文档，测试至少两种不同Embedding的检索效果，对比哪个更符合你的需求，记录下来
   - BGE本地使用：https://github.com/FlagOpen/FlagEmbedding

---

### 第2天：RAG优化技巧

**上午（2h）**
1. (0.5h) 分块策略：固定大小分块 vs 语义分块，各有什么优劣，什么时候用哪种
2. (0.5h) 学习进阶优化：混合检索（向量检索 + 关键词检索），为什么效果更好；重排序：为什么重排序能提高准确率
3. (1h) 学习使用CrossEncoder重排序，看官方示例

**下午（2h）**
1. (2h) **动手实践**：在你上周做的RAG项目上，实践至少一个优化技巧（比如换BGE Embedding，加重排序），对比优化前后效果，记录下来，写在你的项目README里

---

### 第3天：LangChain v1.x核心概念

**上午（2h）**
1. (1h) 理解LangChain是什么：它是大模型应用开发框架，帮你封装了很多通用组件，不用自己重复造轮子
2. (1h) 学习三个核心组件：
   - `PromptTemplate`: 提示词模板复用，避免硬编码
   - `Chain`: 把多个步骤串起来，方便流程管理
   - `Retriever`: 统一检索接口，支持不同向量数据库

**下午（2h）**
1. (2h) **动手**：安装LangChain v1.x，跑通官方的RAG示例，理解每个部分做什么
   - 官方入门：https://python.langchain.com/docs/get_started/introduction

---

### 第4天：LangChain实现完整RAG

**上午（2h）**
1. (2h) 阅读LangChain官方RAG文档，理解官方推荐的最佳实践：https://python.langchain.com/docs/modules/data_connection/

**下午（2h）**
1. (2h) **动手重写**：把你上周做的私人知识库机器人，用LangChain重写一遍，对比手写和用框架区别，理解框架帮你省了多少事

---

### 第5天：工具调用：Function Calling & MCP

**上午（2h）**
1. (1h) 理解核心：什么是工具调用 —— 让LLM能调用外部API/函数，突破自身知识和能力限制，真正能"动手干活"
2. (1h) 学习Function Calling：OpenAI/Anthropic都支持的标准，理解请求响应格式

**下午（2h）**
1. (1h) 学习MCP协议（模型上下文协议）：Anthropic最新推出的统一工具调用标准，理解它解决了"每个模型格式不统一"的问题，一次开发多模型复用
2. (1h) **动手写示例**：用LangChain实现一个简单工具调用，让LLM调用查天气的工具，拿到结果再回答用户，完整跑通

---

### 第6天：两周复盘

**全天（4h）**
1. (2h) 整理两周学习笔记，修复你项目里遇到的bug，完善项目README
2. (1h) 总结：这两周你遇到了哪些坑，你是怎么解决的，记在你的笔记里，后面复习方便
3. (1h) 提交所有代码到GitHub，准备进入第二阶段Agent核心学习
