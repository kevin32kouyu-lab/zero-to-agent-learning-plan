# 0. 前置准备

> 总耗时：4天 × 4小时 = 16小时（如果你已经会Python和Git，可以跳过对应天数）

## 天数拆分

| 天数 | 内容 | 耗时 | 必备？ |
|------|------|------|--------|
| 第1天 | Python基础补全 | 4h | 不会Python必备 |
| 第2天 | Git基础和GitHub使用 | 4h | 不会Git必备 |
| 第3天 | 开发环境搭建 | 4h | 所有人都要做 |
| 第4天 | 确定你的大模型API | 4h | 所有人都要做 |

---

### 第1天：Python基础补全（4h）

**目标**：掌握Agent开发需要的Python最小知识，能写简单脚本

**早上（2h）**：
1. (0.5h) 变量、数据类型、条件判断、循环
2. (0.5h) 函数定义、参数、返回值
3. (0.5h) 类和面向对象基础（理解类、实例、方法）
4. (0.5h) 模块导入、异常处理

**下午（2h）**：
1. (1h) 虚拟环境使用 `venv` / `conda`，学会安装依赖 `pip install`
2. (1h) 做一个小练习：写一个脚本，调用OpenAI API，发送请求，打印返回结果

**推荐资源**：
- 菜鸟教程Python：https://www.runoob.com/python/python-tutorial.html
- 《Python编程：从入门到实践》前8章

---

### 第2天：Git基础和GitHub使用（4h）

**目标**：会用Git管理代码，能推送到GitHub，让别人看到你的项目

**全天（4h）**：
1. (1h) Git基础概念：仓库、提交、分支
2. (1h) 常用命令：`git add` / `git commit` / `git push` / `git pull`
3. (1h) GitHub操作：创建仓库、克隆到本地、提交代码
4. (1h) 练习：创建一个测试仓库，提交一个Python脚本，能在GitHub看到

**推荐资源**：
- GitHub Git手册：https://guides.github.com/introduction/git-handbook/
- B站"Git零基础入门"视频

---

### 第3天：开发环境搭建（4h）

**目标**：把本地开发环境跑起来

**全天（4h）**：
1. (1h) 安装Python 3.10+（推荐用Anaconda管理环境）
2. (1h) 安装VS Code，安装Python插件
3. (1h) 配置国内pip源，加速安装
4. (1h) 测试：创建虚拟环境，安装 `openai` `langchain`，能正常导入

---

### 第4天：确定你的大模型API（4h）

**目标**：有一个可用的大模型API，开发必备

**选项一：OpenAI API**
- 注册地址：https://platform.openai.com/
- 优点：生态最好，所有框架都默认支持，文档完善
- 缺点：需要科学上网，付费

**选项二：通义千问（阿里）**
- 注册地址：https://dashscope.aliyun.com/
- 优点：国内直接用，价格便宜，对中文支持好
- 缺点：Agent生态不如OpenAI完善，但LangChain已经支持

**选项三： Claude （Anthropic）**
- 注册地址：https://console.anthropic.com/
- 优点：上下文窗口大，适合长文档Agent，函数调用做的好
- 缺点：同样需要科学上网

**今天任务**：
1. (2h) 注册至少一个API，拿到API Key
2. (1h) 跑官方示例，测试调用成功
3. (1h) 把API Key配置到环境变量，不要硬编码到代码里（安全好习惯）
