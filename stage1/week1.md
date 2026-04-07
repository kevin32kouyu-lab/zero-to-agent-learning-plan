# 第一阶段 第1周

> 本周目标：建立Agent基础认知，学会API调用和RAG核心原理

| 天 | 上午（2h） | 下午（2h） | 完成打卡 |
|----|------------|------------|----------|
| 第1天 | Agent核心概念入门，理解Agent四大组件 | 阅读热门Agent案例，思考Agent能解决什么问题 | ☐ |
| 第2天 | Prompt工程基础：角色扮演、思维链、少样本学习 | 练习不同Prompt技巧，对比输出效果差异 | ☐ |
| 第3天 | 学习调用LLM API，理解请求/响应格式 | 实现流式输出，写一个简单命令行聊天机器人 | ☐ |
| 第4天 | RAG检索增强生成原理，解决幻觉问题 | 理解RAG完整流程：分块 → Embedding → 检索 → 生成 | ☐ |
| 第5天 | 向量数据库基础，学习Chroma使用 | 将你的PDF文档导入Chroma，实现简单检索 | ☐ |
| 第6天 | 本周项目：做一个简单的私人知识库问答机器人 | 整理笔记，提交项目到GitHub，复盘本周 | ☐ |

---

## 每日详细内容（带具体资源链接，跟着做就行）

### 第1天：Agent核心概念入门

**上午（2h）**
1. (1h) **看视频理解什么是AI Agent**：
   - B站 李宏毅《AI Agent》第一讲：直接搜"李宏毅 AI Agent"，看第一集，通俗易懂
   - 配合读文章：[AI智能体(Agent)保姆级入门指南，零基础小白也能轻松上手](https://zhuanlan.zhihu.com/p/199509730184530938)
2. (1h) 理解记住Agent四大核心组件：
   - 大脑（LLM）：理解需求，做决策
   - 规划：把大目标拆成小步骤
   - 工具：调用外部能力获取信息
   - 记忆：存储历史信息

**下午（2h）**
1. (1h) 看几个实际Agent案例：
   - Devin (Cognition AI): 第一个AI软件工程师，了解Agent能做什么复杂任务
   - AutoGPT: 最早的开源自主Agent，看官网demo
   - Klarna客服Agent: 商业落地案例，了解实际价值
2. (1h) **动手练习**：去Coze（https://www.coze.cn/）体验几个现成Agent，感受一下实际效果，思考：你想用Agent解决什么问题？记下来，后面可以做这个项目

---

### 第2天：Prompt工程基础

**上午（2h）**
1. (0.5h) 看视频学习：【2小时彻底掌握提示词工程】https://www.bilibili.com/video/BV1kZ421s7km/
2. (1.5h) 动手练习核心技巧：
   - 角色扮演：让LLM扮演专业面试官，对比不扮演的输出差异
   - 思维链(CoT): "让我们一步一步思考"，对比直接回答差异
   - 少样本学习：给2-3个例子，让LLM学习格式，测试输出稳定性

**下午（2h）**
1. (1h) 学习结构化Prompt设计，用LangChain `PromptTemplate` 做模板，让LLM输出稳定JSON格式，**代码可复现**
   - 参考代码：https://github.com/microsoft/ai-agents-for-beginners/blob/main/01-prompt-engineering/code/main.py
2. (1h) 学习多轮对话管理：实现简单的历史裁剪，保留最近N轮，节省上下文窗口，写代码练习

---

### 第3天：LLM API调用实战

**上午（2h）**
1. (1h) 打开你选的大模型官方文档（OpenAI/通义千问/Anthropic），看认证方式、请求格式
2. (1h) **动手写代码**：跟着官方示例，写第一个Python脚本，调用API完成一次对话补全，测试成功

**下午（2h）**
1. (1h) 实现流式输出（streaming），让结果一点点出来，用户体验更好，**代码可复现**
   - 参考：https://platform.openai.com/docs/guides/streaming-responses
2. (1h) 封装：把调用封装成一个简单的命令行聊天机器人，支持多轮对话，本地能跑起来

---

### 第4天：RAG检索增强生成原理

**上午（2h）**
1. (1h) 看视频学习：李宏毅《RAG介绍》，或者看这篇文章：[RAG 入门介绍](https://www.zhihu.com/question/592135908/answer/2987353945)
2. (1h) 理解核心：LLM痛点 → 幻觉、知识截止、不知道私有数据；RAG怎么解决：给LLM"开卷考试"，从外部知识库找准确信息再回答

**下午（2h）**
1. (1h) 亲手拆解RAG完整流程，每个步骤做什么：
   1. 文档加载 → 2. 文本分块 → 3. 转成Embedding向量 → 4. 存在向量数据库 → 5. 用户问题检索相关块 → 6. 拼到Prompt给LLM生成答案
2. (1h) 思考：你日常工作哪些场景适合用RAG？记下来，后面可以做项目

---

### 第5天：向量数据库实战

**上午（2h）**
1. (1h) 理解核心：什么是向量Embedding，为什么能用向量相似性搜索找相似文本
2. (1h) **动手安装**Chroma（轻量向量数据库，适合入门），跟着官方文档跑通基础示例：https://docs.trychroma.com/getting-started

**下午（2h）**
1. (1h) **动手实践**：找一个你自己的PDF文档，用 `PyPDF` 加载进来，分块，转成Embedding，存到Chroma
   - 代码参考：https://github.com/microsoft/ai-agents-for-beginners/blob/main/03-rag/code/rag-sample.py
2. (1h) 测试：输入几个问题，检索相关段落，看结果对不对，体验向量检索效果

---

### 第6天：本周项目：简单私人知识库问答机器人

**全天（4h）**
1. (2h) **动手整合**：把前五天内容整合，从文档加载到问答，完整跑通，**代码可运行**
   - 参考完整示例：https://github.com/microsoft/ai-agents-for-beginners/blob/main/03-rag/README.md
2. (1h) 测试，修复bug，优化问答体验，测试几个问题看回答对不对
3. (1h) 写项目README，提交到GitHub，写本周学习笔记总结
