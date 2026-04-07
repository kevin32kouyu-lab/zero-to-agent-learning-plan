# 第三阶段 第5周：多智能体协作开发 - 个人财经助手Agent

> 本周目标：掌握多智能体协作，完成第一个可以写进简历的完整项目：**个人财经助手Agent**
> 
> 项目亮点：输入股票代码/名称，多Agent分工协作，自动拉数据、搜新闻、分析、生成完整投资分析报告

| 天 | 内容 | 时长 | 完成打卡 |
|----|------|------|----------|
| 第1-2天 | 多智能体原理 + 主流框架学习体验 | 4h × 2 | ☐ |
| 第3-4天 | CrewAI/LangGraph/AutoGen 三个框架对比 | 4h × 2 | ☐ |
| 第5天 | 项目需求分析 + 架构设计 | 4h | ☐ |
| 第6-7天 | 开发：数据获取 + 新闻搜索模块 | 4h × 2 | ☐ |
| 第8天 | 开发：用户偏好记忆模块 | 4h | ☐ |
| 第9天 | 开发：Streamlit前端 + 联调 | 4h | ☐ |
| 第10天 | 项目整理 + README + 提交GitHub | 4h | ☐ |
| 第11天 | 代码重构 + 优化文档 | 4h | ☐ |
| 第12天 | 复盘总结 + 休息 | 4h | ☐ |

---

## 👥 第1天：多智能体协作原理

### 上午（2h）：为什么需要多智能体？

### 单个Agent的局限：

1. **能力瓶颈**：一个Agent既要会拿数据又要会分析又要会写报告，什么都做容易什么都不精
2. **缺少评审**：没人给它找错，错了也不知道
3. **复杂度限制**：太复杂的任务，单个Agent规划不过来

### 多智能体优势：

| 优势 | 说明 |
|------|------|
| **专业分工** | 每个Agent专精一件事，做出来质量更高 |
| **互相评审** | 写完别人审稿，错误率大幅下降 |
| **并行处理** | 多个任务可以同时做，节省时间 |
| **灵活扩展** | 要加新能力，加个Agent就行，不用改老代码 |

### 常见协作模式：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **流水线式** | A做完给B，B做完给C，顺序执行 | 数据处理 → 分析 → 报告，清晰的步骤 |
| **层级式** | 一个Manager Agent分给Worker Agents | 大任务拆分，统一调度 |
| **对话式** | 多个Agent自由对话讨论 | 头脑风暴、辩论、需求讨论 |
| **评审式** | 一个写，一个审，通过了就输出 | 代码生成、内容创作 |

我们这个财经助手项目用**流水线+评审**模式，很清晰。

---

### 下午（2h）：主流多Agent框架对比

现在三个主流框架，该怎么选？

| 框架 | 开发方式 | 学习曲线 | 灵活性 | 最适合 |
|------|----------|----------|--------|--------|
| **CrewAI** | 声明式，定义角色和任务 | 简单，几分钟跑通 | 够用 | **快速原型、角色分工明确的项目** |
| **LangGraph** | 代码定义图流程 | 中等 | 非常灵活 | **复杂自定义工作流** |
| **AutoGen** | 对话式多Agent | 中等 | 灵活 | **多Agent自由对话、人类介入** |

**我们的选择：** 第一个项目用 **CrewAI**，简单清晰，快速出成果，容易理解多Agent协作。

---

## 🚀 第2天：CrewAI快速上手

### 上午（2h）：核心概念理解

**CrewAI核心就是三个概念：**

### 1. Agent（智能体）
每个Agent有三个关键属性：
```python
from crewai import Agent

data_collector = Agent(
    role='金融数据收集专家',
    goal='准确获取股票的价格、财报等数据',
    backstory='你有10年金融数据处理经验，擅长从YFinance等API准确提取数据',
    verbose=True,  # 打印详细过程，方便调试
    allow_delegation=False  # 是否允许转派任务给其他Agent
)
```
- `role`：角色，让LLM清楚它是做什么的
- `goal`：目标，它要达到什么效果
- `backstory`：背景故事，给Agent更多上下文，效果更好

### 2. Task（任务）
每个任务分配给一个Agent：
```python
from crewai import Task

collect_data = Task(
    description='获取股票代码 {stock_code} 最近一年股价数据和最新财报数据',
    expected_output='完整JSON格式数据，包含股价、市盈率、市净率、股息率等关键指标',
    agent=data_collector
)
```

### 3. Crew（团队）
把Agent和Task组合成一个团队运行：
```python
from crewai import Crew

finance_crew = Crew(
    agents=[data_collector, analyst],
    tasks=[collect_data, analyze],
    verbose=True
)

# 运行
result = finance_crew.kickoff(inputs={'stock_code': 'AAPL'})
```

就是这么简单！

---

### 下午（2h）：动手跑通官方示例

**安装：**
```bash
pip install crewai
```

**跑通官方示例：** 跟着这篇走：https://docs.crewai.com/getting-started

官方示例是一个"内容创作团队"，有研究员、写手、编辑，协作生成一篇博客文章。

跑通之后你就理解：
- Agent怎么定义
- Task怎么定义
- Crew怎么运行
- 协作过程是什么样的

**常见坑：**
- 需要配置OPENAI_API_KEY环境变量
- CrewAI默认用OpenAI，想用其他模型需要配置`llm`参数

---

## 🌐 第3天：LangGraph 多Agent对话

### 上午（2h）：LangGraph多Agent原理

LangGraph做多Agent，核心是**每个Agent是一个节点，Agent之间通过边传递消息**。

经典的"两个Agent对话"结构：
```
User -> Agent1 -> Agent2 -> Agent1 -> Agent2 -> ... -> 结束
```

每个Agent能看到完整对话历史，自己决定什么时候结束。

**参考官方示例：** https://langchain-ai.github.io/langgraph/tutorials/multi-agent/

---

### 下午（2h）：动手实现简单多Agent对话

**练习：** 实现产品经理和工程师讨论需求

```python
# 伪代码框架
from langgraph.graph import StateGraph

class ConversationState(TypedDict):
    messages: List[BaseMessage]

# 产品经理Agent节点
def pm_agent(state):
    # PM根据对话说下一步需求
    return {"messages": [...]}

# 工程师Agent节点  
def eng_agent(state):
    # 工程师提问/回答技术问题
    return {"messages": [...]}

# 构建图
builder = StateGraph(ConversationState)
builder.add_node("pm", pm_agent)
builder.add_node("eng", eng_agent)
builder.add_edge("pm", "eng")
builder.add_edge("eng", "pm")
builder.set_entry_point("pm")

graph = builder.compile()
```

跑起来，让PM和工程师讨论"做一个财经助手Agent"，看他们聊出什么。

---

## 💬 第4天：AutoGen 多Agent自动对话

### 上午（2h）：AutoGen核心思想

AutoGen是微软推出的多Agent框架，核心特点：
- 多Agent可以**自动对话**
- 支持**人类在环**，人类可以随时介入回答问题
- 支持不同LLM混用

**典型用法：**
- 程序员Agent写代码
- 评审Agent评审代码
- 有问题问人类
- 直到正确才停止

---

### 下午（2h）：跑通AutoGen官方示例

**安装：**
```bash
pip install pyautogen
```

跑通官方示例：两个Agent协作解决数学问题，理解对话流程。

**对比总结（写完这三个你就懂了）：**

| 维度 | CrewAI | LangGraph | AutoGen |
|------|--------|-----------|---------|
| 上手难度 | ⭐ 简单 | ⭐⭐ 中等 | ⭐⭐ 中等 |
| 灵活性 | ⭐⭐ 中等 | ⭐⭐⭐ 很高 | ⭐⭐⭐ 高 |
| 代码量 | 少 | 中等 | 中等 |
| 适合 | 角色分工流水线 | 复杂工作流 | 自由对话协作 |

---

## 📊 第5天：项目 - 个人财经助手Agent 架构设计

### 全天（4h）：需求和架构

### 项目需求

> **输入：** 股票代码或名称（比如"苹果"、"AAPL"、"贵州茅台"、"600519"）
> 
> **输出：** 一份完整的投资分析报告，包含：
> - 最新股价和关键财务指标（PE/PB/股息率/营收增速）
> - 近期相关新闻汇总
> - 趋势分析（近期涨跌原因）
> - 风险提示
> - 投资观点总结

### 角色分工（四个Agent协作）

| Agent角色 | 职责 |
|-----------|------|
| **数据收集Agent** | 从YFinance API拉取股价、财报、关键指标 |
| **新闻搜索Agent** | 搜索这只股票近期新闻，整理要点 |
| **投资分析Agent** | 整合数据和新闻，生成投资分析报告 |
| **审稿Agent** | 检查报告完整性，优化表达，补充风险提示 |

### 完整流程图

```
用户输入股票代码
    ↓
[数据收集Agent] → 输出：财务数据JSON
    ↓
[新闻搜索Agent] → 输出：新闻要点整理
    ↓
[投资分析Agent] → 输出：初稿分析报告
    ↓
[审稿Agent] → 输出：终稿投资分析报告
    ↓
展示给用户
```

### 技术栈

- 框架：CrewAI（简单清晰，适合角色分工）
- 数据API：yfinance（免费，不用key）
- 搜索：SerpAPI（或者Google Custom Search）
- 记忆：Chroma（存储用户投资偏好）
- 前端：Streamlit（简单快速做Web界面）

**画好这个架构图，放到项目README里，面试官一看就清楚。**

---

## 📥 第6天：开发第一部分 - 数据获取模块

### 全天（4h）：对接YFinance API

**YFinance是什么？**
- 免费获取雅虎财经股票数据
- Python库：`yfinance`
- 不用注册，不用API Key，直接用

**安装：**
```bash
pip install yfinance
```

**最小使用示例：**

```python
import yfinance as yf

# 获取苹果公司数据
ticker = yf.Ticker("AAPL")

# 基本信息
info = ticker.info
print("公司名称：", info.get('longName'))
print("行业：", info.get('industry'))
print("当前价格：", info.get('currentPrice'))
print("市盈率(Trailing)：", info.get('trailingPE'))
print("市盈率(Forward)：", info.get('forwardPE'))
print("市净率：", info.get('priceToBook'))
print("股息率：", info.get('dividendYield'))

# 历史股价
history = ticker.history(period="1y")
print(history.head())
```

**你的任务：**
1. 封装一个`DataCollector`类
2. 提供方法：`get_stock_info(symbol)` → 返回标准化JSON数据
3. 至少包含：
   - 基本信息（名称、行业、简介）
   - 当前价格
   - 关键指标（PE、PB、股息率、市值）
   - 近一年股价数据
4. 写测试：测几个股票（AAPL、MSFT、600519.SH），确认数据正确

**注意：**
- 中国A股代码格式：`600519.SH`（上海）、`000001.SZ`（深圳）
- YFinance对A股支持还可以，基本数据都有

---

## 🔍 第7天：开发第二部分 - 新闻搜索模块

### 全天（4h）：实现新闻搜索

**为什么需要新闻搜索？**
- 股价涨跌和近期新闻高度相关
- LLM知识截止，不知道最新新闻
- Agent必须搜了最新新闻才能分析

**选择搜索API：**

| API | 免费额度 | 价格 | 推荐 |
|-----|----------|------|------|
| SerpAPI | 100次/月免费 | $25/月起 | ⭐⭐⭐⭐⭐ 简单，不用自己爬 |
| Google Custom Search | 100次/天免费 | 超量后每千次$5 | ⭐⭐⭐⭐ 便宜 |
| Bing Search | 按月收费 | 性价比不错 | ⭐⭐⭐ |

推荐用SerpAPI，注册就能用，新手友好：https://serpapi.com/

**代码示例：**

```python
from serpapi import GoogleSearch
import os

def search_stock_news(stock_name, num_results=5):
    """搜索股票最新新闻"""
    params = {
        "q": f"{stock_name} 股票 最新新闻 2026",
        "engine": "google_news",
        "api_key": os.getenv("SERPAPI_KEY"),
        "num": num_results
    }
    search = GoogleSearch(params)
    results = search.get_dict()
    news = []
    if "news_results" in results:
        for item in results["news_results"][:num_results]:
            news.append({
                "title": item.get("title"),
                "link": item.get("link"),
                "date": item.get("date"),
                "snippet": item.get("snippet")
            })
    return news

# 测试
print(search_stock_news("苹果公司"))
```

**你的任务：**
1. 封装`NewsSearcher`类
2. 实现`search_latest_news(stock_name)`方法
3. 返回整理好的新闻列表，包含标题、日期、摘要
4. 测试：搜几个股票，看结果对不对

---

## 🧠 第8天：开发第三部分 - 用户偏好记忆

### 全天（4h）：实现长期记忆

**目标：** 让Agent记住你的投资偏好，比如：
> "我是稳健型投资者，更关注分红和估值，不喜欢高波动成长股"

下次你再看股票，Agent会根据你的偏好给出分析。

**实现方案：**
- 用户的偏好描述/历史问答存入Chroma向量库
- 用户新提问时，检索相关偏好
- 把偏好注入Prompt，分析Agent会考虑你的偏好

**代码框架：**

```python
import chromadb
from sentence_transformers import SentenceTransformer

class UserMemory:
    def __init__(self):
        self.client = chromadb.Client()
        self.collection = self.client.create_collection("user_preferences")
        self.model = SentenceTransformer('BAAI/bge-base-zh-v1.5')
    
    def add_preference(self, preference_text):
        """添加用户偏好"""
        self.collection.add(
            documents=[preference_text],
            ids=[f"pref_{len(self.collection.get()['ids'])}"]
        )
    
    def get_relevant_preferences(self, query, top_k=3):
        """检索相关偏好"""
        results = self.collection.query(
            query_texts=[query],
            n_results=top_k
        )
        if results['documents']:
            return "\n".join(results['documents'][0])
        return ""

# 使用
memory = UserMemory()
memory.add_preference("我是稳健型投资者，关注股息率和估值安全边际，厌恶高波动")
pref = memory.get_relevant_preferences("分析贵州茅台")
print(pref)
```

**你的任务：**
1. 实现`UserMemory`类
2. 测试：添加偏好，检索，确认正常工作
3. 在Agent的Prompt里把偏好加进去

---

## 🖥️ 第9天：开发第四部分 - Streamlit前端

### 全天（4h）：做个简单Web界面

**Streamlit是什么？**
- Python写Web界面，几十行代码搞定
- 不用懂HTML/CSS/JS，纯Python

**安装：**
```bash
pip install streamlit
```

**最小财经助手界面：**

```python
import streamlit as st
from crewai import Crew
# 导入你的模块...

st.set_page_config(page_title="个人财经助手Agent", layout="wide")
st.title("📈 个人财经助手Agent")

# 输入框
stock_input = st.text_input("输入股票代码或名称", "AAPL")
user_preference = st.text_area("你的投资偏好（可选）", "我是稳健型投资者，关注长期价值")

if st.button("开始分析"):
    with st.spinner("多Agent协作分析中..."):
        # 1. 收集数据
        data = data_collector.collect(stock_input)
        st.subheader("📊 财务数据")
        st.json(data)
        
        # 2. 搜索新闻
        news = news_searcher.search(stock_input)
        st.subheader("📰 最新新闻")
        for item in news:
            st.markdown(f"- **{item['title']}** - {item['date']}")
            st.write(item['snippet'])
        
        # 3. 运行CrewAI生成报告
        result = finance_crew.kickoff(inputs={...})
        
        st.subheader("📝 投资分析报告")
        st.markdown(result)

# 侧边栏说明
with st.sidebar:
    st.markdown("### 关于")
    st.markdown("基于CrewAI多智能体协作的个人财经助手")
```

**运行：**
```bash
streamlit run app.py --server.port 8501
```
打开 `http://localhost:8501` 就能用了。

**你的任务：**
1. 把前面三个模块集成进来
2. 完整跑通端到端流程：输入 → 数据 → 新闻 → 分析 → 报告展示
3. 修复所有bug，确保本地能正常运行

---

## 📝 第10天：项目整理提交GitHub

### 全天（4h）：整理项目

**项目目录结构（专业一点）：**
```
finance-ai-assistant/
├── .env.example          # 环境变量示例
├── .gitignore            # 忽略.env、__pycache__等
├── README.md             # 项目说明（非常重要！）
├── requirements.txt      # 依赖列表
├── app.py                # Streamlit前端入口
├── agents/
│   ├── __init__.py
│   ├── data_collector.py  # 数据收集Agent
│   ├── news_search.py     # 新闻搜索Agent
│   ├── analyst.py         # 分析Agent
│   └── reviewer.py        # 审稿Agent
├── tools/
│   ├── __init__.py
│   ├── yfinance_tool.py   # YFinance封装
│   ├── search_tool.py     # 搜索封装
│   └── memory.py          # 记忆封装
└── examples/
    └── example_report.md  # 一份示例报告，展示效果
```

**README必须包含这些内容（让你的项目加分）：**

1. **项目介绍**：这个项目做什么的，用了什么技术
2. **架构图**：画一下四个Agent协作流程图（用mermaid，GitHub支持）
3 **快速开始**：一步步说明怎么安装依赖、怎么配置API Key、怎么运行
4. **使用示例**：附上截图或者示例报告链接
5. **效果展示**：展示一份完整的分析报告示例
6. **改进方向**：说说你觉得还可以怎么改进，显示你会思考

**写完推送到GitHub：**
```bash
git init
git add .
git commit -m "initial commit: complete finance ai assistant"
# 关联你的GitHub仓库
git push origin main
```

---

## ✨ 第11天：代码重构和优化

### 全天（4h）：让项目更专业

**做这些事：**

1. **清理代码**（2h）
   - 去掉冗余打印、调试代码
   - 函数和变量命名清晰
   - 每个模块加注释
   - 处理异常：API调用失败怎么办？股票代码不对怎么办？

2. **优化README**（2h）
   - 加上项目截图（运行界面截一张）
   - 检查快速开始步骤对不对，新手按照步骤能跑起来吗？
   - 加上"项目亮点"，写清楚你用到了哪些技术：
     - ✅ 多智能体协作分工，每个Agent负责专一功能
     - ✅ 集成YFinance免费金融数据API
     - ✅ 支持联网搜索最新财经新闻
     - ✅ 用户投资偏好长期记忆
     - ✅ Streamlit简洁Web界面

**加分项：** 做一个示例分析报告，放到`examples/`目录，面试官可以直接看效果。

---

## 📋 第12天：复盘总结

### 全天（4h）：沉淀经验

**要做的事：**

1. **整理开发笔记**（2h）
   - 开发过程中遇到了哪些坑？
   - 你是怎么解决的？
   - 有什么经验教训？
   - 记到你的学习笔记里

2. **项目检查**（1h）
   - 从GitHubclone下来，按照README步骤重新跑一遍，确认能正常运行
   - 别让面试官clone下来跑不起来，印象分大减

3. **休息调整**（1h）
   - 完成了第一个完整项目，很棒！
   - 下周做第二个更有意思的项目

---

## ✅ 项目完成验收标准

你的项目满足这些，就可以写进简历了：

- [ ] 四个Agent分工协作，流程正确
- [ ] 能正确获取股票财务数据
- [ ] 能搜索最新新闻
- [ ] 能记住用户投资偏好
- [ ] Streamlit界面能正常运行
- [ ] GitHub上有完整代码和README
- [ ] README有架构图和使用说明
- [ ] 有示例输出展示效果

---

## 📚 扩展阅读（选看）

- [CrewAI官方文档](https://docs.crewai.com/)
- [LangGraph多Agent示例](https://langchain-ai.github.io/langgraph/tutorials/multi-agent/)
- [AutoGen官方文档](https://microsoft.github.io/autogen/)
- [yfinance文档](https://pypi.org/project/yfinance/)
