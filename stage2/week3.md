# 第二阶段 第3周：ReAct Agent核心 + LangGraph开发

> 本周目标：掌握Agent最核心的ReAct范式，用LangGraph开发完整可运行Agent
> 从这一周开始，你真正开始写Agent了！

| 天 | 上午（2h） | 下午（2h） | 完成打卡 |
|----|------------|------------|----------|
| 第1天 | ReAct模式原理：思考-行动-观察循环 | 手写最简ReAct，理解底层原理 | ☐ |
| 第2天 | Agent三层记忆架构：短期/长期/总结 | 实现简单长期记忆，让Agent记住你 | ☐ |
| 第3天 | 主流框架对比：LangGraph/CrewAI/AutoGen/Dify | 理解各框架适用场景 | ☐ |
| 第4天 | LangGraph核心：节点、边、状态 | 跑通官方入门示例 | ☐ |
| 第5天 | LangGraph实现完整ReAct Agent | 支持工具调用的完整Agent | ☐ |
| 第6天 | 项目改造：RAG + Agent混合架构 | 测试+整理+复盘 | ☐ |

---

## 🧠 第1天：ReAct模式原理 - Agent的心跳

### 上午（2h）：深入理解ReAct

### 什么是ReAct？

ReAct = **Reasoning（思考） + Acting（行动）**，这是目前**最主流**的Agent工作范式。

**核心就是一个循环：**

```
用户问题
    ↓
[思考] LLM分析当前情况，决定下一步做什么
    ↓
[行动] 根据决策调用工具，执行动作（搜索、计算、查库）
    ↓
[观察] 拿到工具返回结果
    ↓
回到[思考]，用新信息继续分析...
    ↓
直到任务完成，输出最终答案
```

### 和CoT（思维链）对比

| 模式 | 特点 | 适用场景 |
|------|------|----------|
| **CoT** | LLM只思考，不行动 | 问答、数学推理，不需要外部信息 |
| **ReAct** | 思考+行动循环，能调用工具 | 需要实时信息、外部数据、实际操作的复杂任务 |

**举个例子：**

> 用户问："昨天特斯拉股价收于多少？"

- CoT：不知道就是不知道，或者瞎编一个（幻觉）
- ReAct："我需要搜索最新股价 → 调用搜索工具 → 拿到结果 → 回答用户"

这就是ReAct的威力：**让LLM能"自己动手"获取需要的信息**。

### ReAct的停止条件

Agent什么时候停止循环？

1.  LLM判断任务已经完成，直接输出答案 → 停止
2.  达到最大步数限制（比如5-10步）→ 停止，防止死循环
3.  用户中断 → 停止

**为什么需要最大步数？** 防止Agent陷入循环思考，浪费token和时间。

---

### 下午（2h）：手写最简ReAct，理解底层

**目标：不用任何Agent框架，手写一个ReAct循环，理解每一步做什么**

只有亲手写一遍，才不会只是"会用框架"，而是真理解。

完整简化代码：

```python
import openai
import json
import os
from dotenv import load_dotenv

load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

# 定义一个简单工具：查询天气（模拟）
def get_weather(city):
    """查询城市天气"""
    mock_data = {
        "北京": "晴天，25℃，西北风3级",
        "上海": "多云，28℃，东南风2级",
        "广州": "小雨，30℃，南风4级"
    }
    return mock_data.get(city, f"未找到{city}的天气信息")

# 工具列表，告诉LLM有哪些工具可用
TOOLS = {
    "get_weather": {
        "description": "查询指定城市的天气",
        "parameters": {
            "city": "城市名称，比如北京"
        },
        "function": get_weather
    }
}

# 构建系统提示词，告诉LLM怎么ReAct
SYSTEM_PROMPT = """
你是一个能调用工具的智能助手。
当你需要回答用户问题时，如果需要外部信息，请按以下格式调用工具：

<|TOOL|>name:{{工具名称}}, args:{{参数JSON}}<|/TOOL|>

如果不需要调用工具，直接回答问题，格式：
<|ANSWER|>{{你的回答}}<|/ANSWER|>

可用工具：
""" + "\n".join([f"- {name}: {info['description']}" for name, info in TOOLS.items()])

def parse_response(response_text):
    """解析LLM回复，判断是回答还是调用工具"""
    if "<|TOOL|>" in response_text and "<|/TOOL|>" in response_text:
        # 提取工具调用部分
        tool_part = response_text.split("<|TOOL|>")[1].split("<|/TOOL|>")[0].strip()
        # 解析 name 和 args
        if "name:" in tool_part and "args:" in tool_part:
            name_part = tool_part.split("args:")[0].replace("name:", "").strip()
            args_part = tool_part.split("args:")[1].strip()
            try:
                args = json.loads(args_part)
                return ("tool", name_part, args)
            except:
                pass
    elif "<|ANSWER|>" in response_text and "<|/ANSWER|>" in response_text:
        answer = response_text.split("<|ANSWER|>")[1].split("<|/ANSWER|>")[0].strip()
        return ("answer", answer, None)
    
    # 如果格式不对，当成回答处理
    return ("answer", response_text, None)

def react_agent(question, max_steps=5):
    """最简ReAct Agent实现"""
    # 对话上下文
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": question}
    ]
    
    for step in range(max_steps):
        print(f"\n--- Step {step + 1}/{max_steps} ---")
        
        # 1. 思考：调用LLM
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=messages,
            temperature=0
        )
        content = response.choices[0].message.content
        print(f"LLM输出: {content[:200]}...")
        
        # 2. 解析输出
        parsed = parse_response(content)
        if not parsed:
            return "解析失败"
        
        type_ = parsed[0]
        
        # 3. 类型判断：回答还是调用工具
        if type_ == "answer":
            answer = parsed[1]
            print(f"\n✅ 任务完成，最终回答：\n{answer}")
            return answer
        
        elif type_ == "tool":
            tool_name = parsed[1]
            tool_args = parsed[2]
            
            print(f"🔧 调用工具: {tool_name} 参数: {tool_args}")
            
            # 4. 执行工具
            if tool_name in TOOLS:
                tool_func = TOOLS[tool_name]["function"]
                observation = tool_func(**tool_args)
                print(f"🔍 工具返回: {observation}")
                
                # 5. 观察：把结果放回上下文，继续循环
                messages.append({
                    "role": "assistant",
                    "content": content
                })
                messages.append({
                    "role": "system",
                    "content": f"工具返回结果: {observation}"
                })
            else:
                observation = f"错误：未找到工具 {tool_name}"
                messages.append({
                    "role": "system",
                    "content": observation
                })
    
    # 超过最大步数
    return f"达到最大步数 {max_steps}，任务未完成"

# 测试
if __name__ == "__main__":
    question = "今天北京天气怎么样？"
    answer = react_agent(question)
```

**运行测试：**
```
python react_simple.py
```

你应该能看到完整的ReAct循环跑起来：LLM判断需要调用工具 → 调用天气工具 → 拿到结果 → 生成回答。

**理解了这个极简版本，再用LangGraph就知道为什么要这么设计了。**

---

## 📝 第2天：Agent记忆管理

### 上午（2h）：三层记忆架构

Agent需要记住东西，不同记忆放在不同地方：

| 记忆类型 | 存储位置 | 作用 | 优缺点 |
|----------|----------|------|--------|
| **短期记忆** | LLM上下文窗口 | 当前对话的最近信息 | ✅ 快，直接用；❌ 容量有限 |
| **长期记忆** | 向量数据库 | 历史对话、用户偏好、知识 | ✅ 容量大，不占窗口；❌ 需要一次检索 |
| **总结记忆** | 摘要后存在上下文/数据库 | 长对话压缩保留核心 | ✅ 平衡容量和信息；❌ 可能丢失细节 |

### 什么时候用什么记忆？

- **短期记忆**：当前会话的最近几轮对话，直接放上下文
- **长期记忆**：跨会话记住用户是谁、用户偏好、之前聊过什么
- **总结记忆**：对话超过10轮后，对前面做摘要，节省空间

---

### 下午（2h）：实现简单长期记忆

**目标：让Agent记住你之前说过的话**

完整实现步骤：

```python
import chromadb
from sentence_transformers import SentenceTransformer

# 初始化Chroma
client = chromadb.Client()
collection = client.create_collection("long_term_memory")
model = SentenceTransformer('BAAI/bge-base-zh-v1.5')

def add_to_memory(user_message, assistant_response):
    """把一轮对话存入长期记忆"""
    memory_text = f"用户: {user_message}\n助理: {assistant_response}"
    # 存在Chroma，自动生成embedding
    collection.add(
        documents=[memory_text],
        ids=[f"mem_{len(collection.get()['ids'])}"]
    )

def retrieve_relevant_memory(query, top_k=3):
    """检索和当前query相关的历史记忆"""
    results = collection.query(
        query_texts=[query],
        n_results=top_k
    )
    if results['documents']:
        return "\n---\n".join(results['documents'][0])
    return ""

# 在Agent调用LLM前，注入相关记忆：
def agent_with_long_term_memory(user_query):
    # 1. 检索相关记忆
    memory = retrieve_relevant_memory(user_query)
    
    # 2. 把记忆放进Prompt
    prompt = f"""以下是之前的相关对话记忆：
{memory}

请基于记忆回答用户现在的问题：{user_query}
"""
    
    # 3. 调用LLM得到回答
    answer = llm(prompt)
    
    # 4. 把这轮对话存入记忆
    add_to_memory(user_query, answer)
    
    return answer
```

**测试：**
1.  说："我喜欢蓝色，不喜欢红色" → 存入记忆
2.  问："我喜欢什么颜色？" → 能从记忆里找到正确答案 → 成功

**常见问题：**
- Q: 检索出来的记忆放哪里？
- A: 放在系统提示词后面，用户问题前面，LLM能看到

---

## 🔀 第3天：主流Agent框架对比

### 上午（2h）：各框架选型分析

目前主流框架该怎么选？一张表看懂：

| 框架 | 开发方式 | 最适合场景 | 优点 | 缺点 | 推荐指数 |
|------|----------|------------|------|------|----------|
| **LangGraph** | 代码开发 | 自定义复杂Agent、复杂工作流 | 状态管理强大，灵活，生态好，LangChain原生支持 | 需要写代码，入门稍慢 | ⭐⭐⭐⭐⭐ |
| **CrewAI** | 代码开发 | 多智能体协作项目 | 角色分工清晰，API简洁，几分钟跑通 | 复杂流程自定义不够灵活 | ⭐⭐⭐⭐ |
| **AutoGen** | 代码开发 | 多智能体对话，支持人类介入 | 微软官方，成熟，支持人类在环 | 适合对话场景，固定流程较重 | ⭐⭐⭐⭐ |
| **Dify** | 低代码/可视化 | 快速产品上线、业务Agent | RAG/Agent/发布一条龙，一键发布 | 高度自定义受限 | ⭐⭐⭐⭐ |
| **LlamaIndex** | 代码开发 | RAG为主的应用 | RAG生态丰富 | Agent编排不如LangGraph | ⭐⭐⭐ |

### 选型决策树（直接用）

```
你要做什么？
    ↓
快速做个能用的产品上线？
    → 是 → 选 Dify
    → 否 ↓  
需要多个Agent分工协作？
    → 是 → 选 CrewAI
    → 否 ↓
需要自定义复杂工作流？
    → 是 → 选 LangGraph（推荐现在学习这个）
    → 否 → 看具体场景
```

**本教程为什么选LangGraph？**
- 自定义灵活，能满足大部分开发需求
- 现在行业 adoption 最快，求职找工作也用得到
- 理解了LangGraph，其他框架一看就会

---

### 下午（2h）：动手体验各框架

任务：每个框架都跑一遍官方Hello World，感受开发体验

```bash
# LangGraph
pip install langgraph
# 跑通：https://langchain-ai.github.io/langgraph/tutorials/introduction/

# CrewAI
pip install crewai
# 跑通：https://docs.crewai.com/getting-started

# Dify
# 用docker跑，或者用云版本体验：https://dify.ai/
```

体验完你就知道为什么我们选LangGraph当主线学习了。

---

## 🌐 第4天：LangGraph核心概念

### 上午（2h）：理解LangGraph设计

LangGraph = LangChain + 图状编排，专门解决Agent的**状态管理**和**循环控制**问题。

三大核心概念：

### 1. **节点（Node）**
- 每个节点只做一件事情
- 比如：`call_llm`节点调用LLM，`tool_node`执行工具调用
- 你可以自定义任意节点，实现自定义逻辑

**示例节点：**
```python
def call_llm_node(state):
    # 从state读消息
    messages = state["messages"]
    # 调用LLM
    response = llm.invoke(messages)
    # 返回更新state
    return {"messages": [response]}
```

### 2. **边（Edge）**
- 定义节点之间的流转关系
- 普通边：固定 A → B
- **条件边**：根据state判断下一步走哪里（Agent的关键）

**示例条件边：**
```python
def should_continue(state):
    last_message = state["messages"][-1]
    # 如果LLM说要调用工具 → 去工具节点
    if last_message.tool_calls:
        return "continue"
    # 否则 → 结束，输出答案
    return "end"
```

### 3. **State（状态）**
- 整个图运行过程中，所有数据存在State里
- 所有节点都能读写State
- 你可以定义State的结构，比如：
```python
class AgentState(TypedDict):
    messages: List[BaseMessage]  # 对话历史
    intermediate_steps: List     # 中间步骤记录
```

**一句话总结LangGraph：** 你定义节点（做什么）、边（怎么流转）、状态（数据存在哪），LangGraph帮你运行。

---

### 下午（2h）：跑通官方入门示例

跟着官方教程走一遍：https://langchain-ai.github.io/langgraph/tutorials/introduction/

你需要跑通：
1.  最简单的聊天机器人
2.  带条件分支的机器人
3.  理解状态怎么在节点间传递

完成这步，LangGraph核心概念你就掌握了。

---

## ⚙️ 第5天：LangGraph实现完整ReAct Agent

### 上午（2h）：学习官方示例

官方ReAct Agent教程：https://langchain-ai.github.io/langgraph/tutorials/react-agent/

重点理解：
1.  状态定义：需要存哪些信息？
2.  工具节点：怎么执行工具调用？
3.  条件判断：怎么决定继续还是停止？
4.  编译图：怎么把节点边编成可运行的图？

---

### 下午（2h）：动手写ReAct Agent

最简LangGraph ReAct Agent骨架：

```python
from typing import TypedDict, Annotated, Sequence
import operator
from langchain_core.messages import BaseMessage
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langgraph.prebuilt import ToolNode

# 1. 定义状态
class AgentState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], operator.add]

# 2. 定义工具
@tool
def get_weather(city: str) -> str:
    """查询城市天气"""
    # 实现略...
    return f"{city} 晴天 25度"

tools = [get_weather]
tool_node = ToolNode(tools)

# 3. LLM节点：绑定工具
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
llm_with_tools = llm.bind_tools(tools)

def call_llm(state):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

# 4. 条件判断：是否继续调用工具
def should_continue(state):
    last_message = state["messages"][-1]
    # 如果有工具调用，继续执行工具
    if last_message.tool_calls:
        return "continue"
    # 否则结束
    return "end"

# 5. 构建图
workflow = StateGraph(AgentState)

# 添加节点
workflow.add_node("agent", call_llm)
workflow.add_node("tools", tool_node)

# 添加边
workflow.set_entry_point("agent")  # 入口
workflow.add_conditional_edges(
    "agent",
    should_continue,
    {
        "continue": "tools",
        "end": END
    }
)
workflow.add_edge("tools", "agent")  # 工具执行完回到agent

# 6. 编译运行
app = workflow.compile()

# 7. 测试
from langchain_core.messages import HumanMessage
inputs = {"messages": [HumanMessage(content="北京天气怎么样？")]}
for output in app.stream(inputs):
    for node, output_state in output.items():
        print(f"Node: {node}")
        print(output_state)
```

运行起来，你就得到了一个**完整的ReAct Agent**，支持任意工具扩展。

**测试要点：**
- 问题需要工具 → Agent正确调用工具
- 工具返回后 → Agent正确生成回答
- 不需要工具 → Agent直接回答

---

## 🚀 第6天：项目改造 - RAG + Agent混合架构

### 全天（4h）：改造你的RAG项目

**任务：** 把你之前做的RAG项目改造成**RAG + ReAct Agent**混合架构

改造后的工作流程：

```
用户问题
    ↓
Agent判断：知识库⾥有没有？
    ↓         ↓
   有直接回答   需要搜索 → 调用检索工具找相关文档
    ↓         ↓
         整合信息 → 回答用户
```

**优势：**
- 比纯RAG更智能：Agent可以判断什么时候需要检索，需要检索几次
- 可以结合多个工具：比如先搜知识库，不知道再搜谷歌

**改造步骤：**

1. **把RAG封装成工具**
```python
@tool
def search_knowledge(query: str) -> str:
    """从私人知识库搜索相关信息.
    
    Args:
        query: 搜索关键词或问题
    """
    # 调用你的向量库检索，返回相关段落
    results = retriever.get_relevant_documents(query)
    return "\n\n---\n\n".join(doc.page_content for doc in results)
```

2. **把这个工具加到ReAct Agent里**

3. **测试**：问几个问题，看Agent会不会正确调用知识库工具

4. **整理代码**，写README，提交GitHub

**验收标准：**
- [ ] Agent能正确判断什么时候需要调用知识库
- [ ] 能正确检索并基于检索结果回答
- [ ] 不需要检索的时候能直接回答
- [ ] 代码推送到GitHub，有使用说明

---

## ✅ 本周学习检查清单

学完本周，你应该掌握：

- [ ] 能解释ReAct范式的原理，说出思考-行动-观察循环
- [ ] 能手写极简ReAct，理解底层循环
- [ ] 知道Agent三层记忆各自用途，能实现简单长期记忆
- [ ] 能说出主流Agent框架的选型，知道什么时候选什么
- [ ] 理解LangGraph三大核心：节点、边、状态
- [ ] 能用LangGraph写出完整的ReAct Agent
- [ ] 完成RAG+Agent混合项目，代码可运行

---

## 📝 本周测试题（带答案）

<details>
<summary>点击展开 ▶️ 1. ReAct范式的核心是什么？</summary>

> A. 只思考不行动  
> B. 思考 → 行动 → 观察循环  
> C. 一次生成完整答案  
> D. 搜索 → 思考 → 输出  

**答案：B**  
解释：ReAct = Reasoning + Acting，核心就是循环思考-行动-观察，直到任务完成。
</details>

<details>
<summary>点击展开 ▶️ 2. Agent为什么需要最大步数限制？</summary>

> A. 加快速度  
> B. 节省token，防止死循环  
> C. 减少代码长度  
> D. 提高准确率  

**答案：B**  
解释：LLM可能陷入无限循环思考，最大步数防止浪费token和死循环。
</details>

<details>
<summary>点击展开 ▶️ 3. LangGraph的三大核心概念是？</summary>

> A. 节点、边、状态  
> B. 输入、输出、模型  
> C. Agent、工具、记忆  
> D. Prompt、LLM、输出  

**答案：A**  
解释：LangGraph基于图编排，核心就是节点（做什么）、边（怎么走）、状态（数据存在哪）。
</details>

<details>
<summary>点击展开 ▶️ 4. 什么是条件边？</summary>

> A. 所有条件都满足才能走  
> B. 根据上一个节点输出，决定下一步走哪个节点  
> C. 必须满足条件才能继续  
> D. 多个条件并行  

**答案：B**  
解释：ReAct中"需要工具调用去工具节点，不需要就结束"就是条件边的典型应用。
</details>

<details>
<summary>点击展开 ▶️ 5. Agent三层记忆中，长期记忆存在哪里？</summary>

> A. LLM上下文窗口  
> B. 向量数据库  
> C. 存在硬盘文件不检索  
> D. LLM权重里  

**答案：B**  
解释：长期记忆存在向量数据库，用到的时候检索相关内容注入上下文，不占LLM上下文。
</details>

<details>
<summary>点击展开 ▶️ 6. RAG + Agent混合架构，RAG封装成什么给Agent用？</summary>

> A. 工具  
> B.  Prompt  
> C. 状态  
> D. 节点  

**答案：A**  
解释：Agent通过工具调用使用RAG，需要的时候就调用知识库检索，比固定检索更灵活。
</details>

<details>
<summary>点击展开 ▶️ 7. 代码题：补全LangGraph条件判断</summary>

```python
def should_continue(state):
    last_message = state["messages"][-1]
    if ____①____:
        return "continue"
    else:
        return "end"
```

**① 应该填什么？**

**答案：** `last_message.tool_calls`

**解释：** 如果最后一条消息有工具调用，就继续去工具节点执行，否则结束输出答案。
</details>

---

## 📚 扩展阅读（选看）

- [ReAct论文原文](https://arxiv.org/abs/2210.03629)
- [LangGraph官方文档](https://langchain-ai.github.io/langgraph/)
- [LangGraph Recipes](https://github.com/langchain-ai/langgraph/tree/main/examples) - 大量官方示例
