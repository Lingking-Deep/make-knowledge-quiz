# Cline Skills

> 复用型 Agent 操作手册集合 / Skill Runtime。

本仓库以 **Agent Skill** 的形式存放可被 Agent 自动发现并按需加载的能力模块。每一个 Skill 是一个文件夹，其中以 `SKILL.md` 作为核心（YAML frontmatter 声明名称与描述，正文写执行指令），并可附带 agents/<runtime>.yaml` 作为特定推理引擎的配置。

- 📦 技能注册表：见 skills/ 目录，每个子目录即为一个技能；
- 🤖 Agent 配置：`agents/openai.yaml` 等，定义使用 OpenAI 兼容模型时的系统指令、模型与参数；
- 🛠️ 校验：后续通过 quick_validate.py 进行多类型文本回归校验。

---

## 技能列表

| 技能 | 说明 | 状态 |
| --- | --- | --- |
| make-knowledge-quiz | 读取用户粘贴的文本 / 本地 TXT 或 Markdown，基于原文生成 5～8 道自测题（单选+判断）。所有考点、选项、解析均来自原文，禁止编造。 | ✅ 正常 |

---

## 1. 项目解决什么问题

make-knowledge-quiz 技能解决的是「基于给定文本快速生成自测考题」的问题。

在阅读文章、文档或 Markdown 笔记后，用户常需要验证自己是否理解了核心知识点，但手动出题耗时且容易遗漏要点。该技能读取用户粘贴的文本或本地 .txt/.md 文件，仅从原文中提取有效考点，生成结构化的单选题与判断题，供知识点自检。

核心边界：**所有考点、选项、解析必须全部来自输入原文，严禁编造或引入外部知识**；若原文有效考点不足 2 个，直接提示无法生成，不强行凑数。

---

## 2. 主要功能

- 接收用户粘贴的纯文本 / Markdown 内容，或读取本地 .txt / .md 文件内容；
- 从原文中提取有效知识点，合并重复考点，同一知识点不出多道题；
- 生成 2～8 道自测题（目标 5～8 道），单选题为主，可少量包含判断题；
- 输出固定 Markdown 结构：题目序号、题型、题干、选项（单选才有）、答案、考点解析；
- 原文考点不足时（少于 2 个有效考点），回复：当前文本可提取的有效考点不足，无法生成自测题；
- 不额外总结原文，不输出知识卡片，只输出结构化自测题。

### 拒绝场景
- 用户传入 PDF 链接、网页链接、图片、Anki 文件：提示暂不支持该类型输入；
- 用户要求超原文拓展知识点、出超纲题目：拒绝并说明限制。

---

## 3. 安装方法

本技能为 **Agent Skill**（Agent 操作手册），不含可执行脚本，无需通过包管理器安装。

1. 将 skills/make-knowledge-quiz/ 目录（含 SKILL.md 与 `agents/openai.yaml`）放入 Agent 的技能注册目录；
2. Agent 启动时会自动扫描并发现该技能，无需额外配置或安装依赖；
3. 无需初始化 Git，无需上传 GitHub。

项目文件布局：

```text
skills/make-knowledge-quiz/
├── SKILL.md                 # 技能说明书（约束、输入、输出格式、拒绝场景）
└── agents/
    └── openai.yaml          # OpenAI 兼容 Agent 配置（模型、指令、参数）
README.md                    # 项目根目录说明文件
```

---

## 4. 使用方法

1. 将原文文本直接粘贴给 Agent，或上传本地 .txt / .md 文件内容；
2. Agent 识别到与知识自测相关的任务后，自动加载 make-knowledge-quiz 技能；
3. Agent 按照 `agents/openai.yaml` 中的系统指令，仅从原文提取考点并生成自测题；
4. 若原文有效考点不足，Agent 会直接回复提示语，不会强行生成题目；
5. （可选）多类型文本测试优化后，运行 quick_validate.py 进行回归校验。

---

## 5. 输入输出示例

### 输入示例

用户粘贴以下文本：

> Agent-to-Agent (A2A) 协议最初由 Google 于 2025 年 4 月正式发起，并于同年 6 月捐赠给了 Linux 基金会。A2A 协议属于横向协作，连接对象是智能体与智能体（跨网络、跨框架）；而 MCP 协议属于纵向集成，连接对象是智能体与工具、数据库、本地文件系统。A2A 任务状态机包含 8 种状态，其中 TASK_STATE_COMPLETED、TASK_STATE_FAILED、TASK_STATE_CANCELED、TASK_STATE_REJECTED 为终态；TASK_STATE_SUBMITTED、TASK_STATE_WORKING、TASK_STATE_INPUT_REQUIRED、TASK_STATE_AUTH_REQUIRED 为非终态。智能体名片（Agent Card）默认托管在 /.well-known/agent.json 路径下。

### 输出示例

```markdown
### 题目1
【题型】判断
题干：A2A 协议最初由 Google 于 2025 年 4 月正式发起，并于同年 6 月捐赠给了 Linux 基金会。
✅ 答案：正确
📝 考点解析：原文提到 A2A 协议最初由 Google 于2025年4月正式发起，同年6月捐赠给了 Linux 基金会。

### 题目2
【题型】单选
题干：根据原文，A2A 协议与 MCP 协议在协作轴向上的核心区别是什么？
选项：
A. A2A 属于纵向集成，连接对象是智能体与工具；MCP 属于横向协作，连接对象是智能体与智能体
B. A2A 属于横向协作，连接对象是智能体与智能体；MCP 属于纵向集成，连接对象是智能体与工具、数据库、本地文件系统
C. A2A 与 MCP 都属于纵向集成
D. A2A 与 MCP 都属于横向协作
✅ 答案：B
📝 考点解析：原文对比表明，A2A 为横向协作，连接对象为智能体与智能体；MCP 为纵向集成，连接对象为智能体与工具、数据库、本地文件系统。

### 题目3
【题型】单选
题干：根据 A2A 标准规范，智能体名片（Agent Card）默认应当托管在哪个路径下？
选项：
A. /agent-card.json
B. /.well-known/agent.json
C. /a2a/agent.json
D. /.well-known/a2a.json
✅ 答案：B
📝 考点解析：原文说明 Agent Card 默认放置在网络域名的知名路径下，即 https://{domain_name}/.well-known/agent.json。

### 题目4
【题型】判断
题干：TASK_STATE_WORKING 属于 A2A 任务状态机中的终态。
✅ 答案：错误
📝 考点解析：原文指出终态包括 TASK_STATE_COMPLETED、TASK_STATE_FAILED、TASK_STATE_CANCELED、TASK_STATE_REJECTED；TASK_STATE_WORKING 为非终态活跃状态。

### 题目5
【题型】单选
题干：以下哪项不是原文中提到的 A2A 协议支持的通信传输契约之一？
选项：
A. HTTP/HTTPS + JSON-RPC 2.0
B. HTTP + REST/JSON
C. gRPC
D. MQTT
✅ 答案：D
📝 考点解析：原文列举的四种契约为 HTTP/HTTPS + JSON-RPC 2.0、HTTP + REST/JSON、gRPC、WebSockets，未提及 MQTT。

### 题目6
【题型】单选
题干：A2A 协议中，允许智能体在不暴露自身内部 prompt、内部记忆、底层调用工具链的前提下接收任务并输出结果，这一原则称为？
选项：
A. 任务状态机原则
B. 不透明性保护（Opacity Preservation）
C. 服务发现原则
D. 无状态任务模型
✅ 答案：B
📝 考点解析：原文指出 A2A 协议设计哲学中最核心的一条原则是不透明性保护（Opacity Preservation）。

### 题目7
【题型】单选
题干：在 A2A 协议的 HTTP/HTTPS + JSON-RPC 2.0 契约中，用于客户端查询特定任务最新状态和进展的 API 方法是？
选项：
A. message/send
B. tasks/cancel
C. tasks/get
D. tasks/submit
✅ 答案：C
📝 考点解析：原文说明 tasks/get 用于客户端查询特定任务的最新状态和进展。

### 题目8
【题型】判断
题干：A2A Client（客户端智能体/委托方）的核心职责包括服务发现、契约检验、任务分发和状态跟进。
✅ 答案：正确
📝 考点解析：原文列出 A2A Client 的核心职责为服务发现、契约检验、任务分发、状态跟进。
```

---

## 限制

暂不支持：网页抓取、PDF、Anki、图形界面、文件以外的媒体资源。
