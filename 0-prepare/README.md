# 0. 前置准备

> 总耗时：4天 × 4小时 = 16小时（如果你已经会Python和Git，可以跳过对应天数）
>
> 这里放了**最小必要的核心知识点**，你不用再去找其他资料，看完就能开工。

## 天数拆分

| 天数 | 内容 | 耗时 | 必备？ |
|------|------|------|--------|
| 第1天 | Python基础补全 | 4h | 不会Python必备 |
| 第2天 | Git基础和GitHub使用 | 4h | 不会Git必备 |
| 第3天 | 开发环境搭建 | 4h | 所有人都要做 |
| 第4天 | 确定你的大模型API | 4h | 所有人都要做 |

---

## 第1天：Python基础补全（4h）

**🎯 目标**：掌握Agent开发需要的Python最小知识，能写简单可运行脚本

### 早上（2h）核心知识点
你只需要学会这些，足够开发Agent了：

1. **(0.5h) 基础语法**
   - 变量和数据类型：`int` `str` `list` `dict` `bool`
   - 条件判断：`if ... elif ... else`
   - 循环：`for` 循环遍历列表字典，`while` 循环
   - 切片操作：`list[0:5]`，会用这个很方便

   ✅ 练习题：
   ```python
   # 定义一个列表，循环打印每个元素
   # 写一个判断奇偶的函数
   ```

2. **(0.5h) 函数**
   - 函数定义：`def function_name(param):`
   - 参数、默认参数、返回值 `return`
   - 匿名函数 `lambda`，了解就行

   ✅ 练习题：
   ```python
   # 写一个函数，输入两个数，返回它们的和
   ```

3. **(0.5h) 面向对象基础**
   - 类和实例：`class ClassName:`，`obj = ClassName()`
   - 实例方法：`def method(self, ...):`
   - `__init__` 构造方法，了解作用

   ✅ 练习题：
   ```python
   # 定义一个ChatBot类，有name属性，有reply方法
   ```

4. **(0.5h) 模块和错误处理**
   - 模块导入：`import module` `from module import something`
   - 异常处理：`try ... except ...`，知道怎么抓错误

### 下午（2h）环境和练习
1. **(1h) 虚拟环境和包管理**
   - `venv` 创建虚拟环境：
     ```bash
     python -m venv myenv
     # windows激活
     myenv\Scripts\activate
     ```
   - `conda` 用得更多也可以，一样用法
   - `pip install` 安装包，`pip list` 看已装包
   - 学会用 `requirements.txt` 保存依赖：`pip freeze > requirements.txt`

2. **(1h) 动手练习**
   > 写一个最简单的API调用脚本，跑通就会了
   ```python
   import os
   from openai import OpenAI

   client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
   response = client.chat.completions.create(
       model="gpt-3.5-turbo",
       messages=[{"role": "user", "content": "你好!"}]
   )
   print(response.choices[0].message.content)
   ```
   跑通这个，你Python基础就够了！

**推荐资源**：
- 菜鸟教程Python：https://www.runoob.com/python/python-tutorial.html
- 《Python编程：从入门到实践》前8章（更系统）

---

## 第2天：Git基础和GitHub使用（4h）

**🎯 目标**：会用Git管理代码，能推送到GitHub，代码备份分享都方便

### 全天（4h）

1. **(1h) 核心概念**
   - `仓库(repository)`：存代码的地方，本地+远程
   - `commit`：提交，保存代码快照
   - `branch`：分支，开发互不干扰
   - `push/pull`：同步本地和远程代码

2. **(1h) 常用命令必须会**
   ```bash
   git init  # 初始化本地仓库
   git add .  # 添加所有文件到暂存区
   git commit -m "commit message"  # 提交
   git remote add origin <url>  # 关联远程仓库
   git push -u origin main  # 推送到远程
   git pull  # 拉取远程更新
   git log  # 看提交历史
   ```

3. **(1h) GitHub操作**
   - 注册GitHub账号（免费就行）
   - 创建新仓库
   - 把本地仓库推送到GitHub
   - 在GitHub网页看你提交的代码

4. **(1h) 练习**
   - 创建一个叫 `test` 的仓库，提交刚才写的API调用脚本，能在GitHub网页看到就ok

**推荐资源**：
- GitHub Git官方手册：https://guides.github.com/introduction/git-handbook/
- B站搜索"Git零基础入门"，找个1小时以内的视频看，上手很快

---

## 第3天：开发环境搭建（4h）

**🎯 目标**：本地开发环境完全跑起来

### 全天（4h）

1. **(1h) 安装Python**
   - 下载地址：https://www.python.org/downloads/
   - 推荐安装 **Python 3.10+**，不要太老版本
   - Windows安装记得勾 `Add Python to PATH`

2. **(1h) 安装VS Code**
   - 下载地址：https://code.visualstudio.com/
   - 安装 **Python插件**（微软官方那个）
   - 学会：打开文件夹、运行python脚本、debug断点调试

3. **(1h) 配置国内pip源（加速下载）**
   - 创建 `%APPDATA%\pip\pip.ini` (Windows)，写入：
     ```ini
     [global]
     index-url = https://pypi.tuna.tsinghua.edu.cn/simple
     ```
   - 以后安装包速度会快很多

4. **(1h) 验证测试**
   ```bash
   # 创建虚拟环境
   python -m venv agent-env
   agent-env\Scripts\activate
   # 安装依赖
   pip install openai langchain chromadb
   # python里导入，不报错就成功了
   python -c "import openai, langchain, chromadb; print('OK')"
   ```
   不报错就搞定了！

---

## 第4天：确定你的大模型API

**🎯 目标**：有一个可用的大模型API，开发必备，这里给你对比，选一个适合你的

| 选项 | 厂商 | 国内访问 | 价格 | 优点 | 缺点 |
|------|------|----------|------|------|------|
| OpenAI API | OpenAI | 需要科学上网 | 按token收费，便宜 | 生态最好，所有框架支持，文档完善 | 需要科学上网 |
| 通义千问 | 阿里 | 直接访问 | 非常便宜 | 中文支持好，速度快 | Agent生态稍弱，但LangChain支持 |
| Claude | Anthropic | 需要科学上网 | 合理 | 上下文窗口超大，适合长文档，工具调用做的好 | 需要科学上网 |
| 豆包 | 字节 | 直接访问 | 免费额度够用 | 国内直接用，价格便宜 | Agent生态还在完善 |

### 今天任务（4h）
1. **(2h)** 选一个，注册账号，拿到API Key
2. **(1h)** 跑官方给的示例，测试调用成功，能返回正确回答
3. **(1h)** 安全好习惯：把API Key存在**环境变量**，不要硬编码写到代码里（避免上传GitHub泄露key）
   - Windows：`set OPENAI_API_KEY=your_key`
   - 或者用 `python-dotenv`，从 `.env` 文件加载，更方便
