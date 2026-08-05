<div align="center">

# 🤖 My First Agent

**不依赖任何 Agent 框架，从零手写 LLM Agent —— 理解原理，而非调包**

[![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python&logoColor=white)](https://www.python.org/)
[![OpenAI SDK](https://img.shields.io/badge/OpenAI%20SDK-compatible-412991?logo=openai&logoColor=white)](https://platform.moonshot.cn/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](https://github.com/)

</div>

---

## 📖 项目简介

这是我在学习大模型应用开发期间的实践项目。市面上大多数教程直接教 LangChain 等框架，但框架把 Agent 的核心机制都藏了起来。本项目反其道而行：**只用最基础的 OpenAI 兼容 SDK，手写 Agent 的每一个零件**——消息管理、上下文窗口、Agent Loop、工具调用、异常兜底。

> 写这个项目的目的：当面试官问"Agent 的原理是什么"时，我回答的不是"我会用某某框架"，而是"我写过它的每一行"。

## ✨ 功能特性

| 模块 | 文件 | 实现的功能 |
|------|------|-----------|
| 💬 多轮对话 | `多轮对话.py` | system 角色设定 · 多轮记忆 · Token 消耗统计 · 滑动窗口 · 流式输出 · 会话持久化 · 成本估算 |
| 🛠️ 手写 Agent | `agent.py` | 工具定义（JSON Schema）· Agent Loop · Function Calling · 工具结果回传 · 死循环防护 · 异常兜底 |

**核心亮点**

- 🧠 **真正的 Agent Loop**：模型自主决定"直接回答"还是"调用工具"，可多轮连续调用
- 🔧 **Function Calling 全流程**：工具定义 → 模型请求 → 本地执行 → 结果回传 → 组织回答
- 🪟 **上下文管理**：滑动窗口防止历史超限，直观演示"模型为什么会失忆"
- 🛡️ **工程化防护**：最大循环次数限制、异常时消息回滚、API Key 环境变量读取

## 🏗️ 核心原理：Agent Loop

```
┌──────────────────────────────────────────────┐
│                Agent Loop                    │
│                                              │
│   用户输入                                   │
│      │                                       │
│      ▼                                       │
│   追加进 messages ─────────────────────┐     │
│      │                                │     │
│      ▼                                │     │
│   发送给 LLM（连同 tools 定义）        │     │
│      │                                │     │
│      ├── 返回纯文本 ──→ 输出回答，结束  │     │
│      │                                │     │
│      └── 返回 tool_calls              │     │
│             │                         │     │
│             ▼                         │     │
│        本地执行工具函数                 │     │
│             │                         │     │
│             ▼                         │     │
│        结果以 role="tool" 回填 ────────┘     │
│        （最多循环 MAX_LOOPS 次，防死循环）   │
└──────────────────────────────────────────────┘
```

**关键认知**：LLM 只负责决策"是否调工具、传什么参数"，真正的执行永远由本地代码完成。Agent = LLM（大脑）+ 工具（手脚）+ messages（记忆）+ 循环（自主决策）。

## 🚀 快速开始

**1. 克隆仓库**

```bash
git clone https://github.com/你的用户名/my-first-agent.git
cd my-first-agent
```

**2. 安装依赖**

```bash
pip install openai
```

**3. 配置 API Key**（以 Moonshot 为例，任何 OpenAI 兼容平台均可）

```bash
# Mac / Linux
export MOONSHOT_API_KEY="你的API Key"

# Windows PowerShell
$env:MOONSHOT_API_KEY="你的API Key"
```

**4. 运行**

```bash
# 体验多轮对话
python 多轮对话.py

# 体验手写 Agent
python agent.py
```

## 🎬 运行演示

**多轮记忆 + 上下文管理**

```
你: 我叫小明，今年20岁
助手: 你好小明！很高兴认识你～
  [Token] 输入 45 + 输出 12 = 共 57

你: 我叫什么名字？
助手: 你叫小明呀！
  [Token] 输入 78 + 输出 8 = 共 86     ← Token 随历史增长（无状态 API）
```

**Agent 工具调用**

```
你: 23*47等于多少？
  [Agent] 模型请求调用工具 calculator，参数：{'expression': '23*47'}
  [Agent] 工具返回：1081
助手: 23 乘以 47 等于 1081。

你: 我还剩多少话费？余额除以2是多少？
  [Agent] 模型请求调用工具 query_balance，参数：{}
  [Agent] 工具返回：85.3元
  [Agent] 模型请求调用工具 calculator，参数：{'expression': '85.3/2'}
  [Agent] 工具返回：42.65
助手: 您当前余额 85.3 元，除以 2 等于 42.65。   ← 连续调用两个工具！
```

## 📁 项目结构

```
my-first-agent/
├── 多轮对话.py      # 第一阶段：多轮对话程序
├── agent.py        # 第二阶段：手写 Agent（工具调用）
├── README.md       # 项目说明
└── chat_history.json   # 运行时自动生成的会话存档
```

## 🧪 内置实验（每个都对应一个大模型概念）

| 操作 | 验证的概念 |
|------|-----------|
| 第 1 轮告知姓名，第 3 轮询问 | 多轮记忆 = 客户端重发完整历史（无状态 API） |
| 输入 `/clear` 后再问姓名 | 记忆全在 messages 里，模型本身无状态 |
| 连续对话 11+ 轮再问早期内容 | 滑动窗口 / 上下文管理 |
| 删除 `tools` 参数后问计算题 | Agent vs 纯 LLM：有无工具的差异（幻觉对照实验） |
| 观察每轮 Token 打印 | Token 计费，成本随轮数线性增长 |

## 🛣️ 学习路线图

- [x] 多轮对话程序（Token / Prompt / 上下文窗口）
- [x] 手写 Agent（Agent Loop / Function Calling）
- [ ] 接入真实 API 工具（天气 / 汇率）
- [ ] 工具参数校验与更完善的异常处理
- [ ] 简易 RAG：外挂文档知识库，对比幻觉改善
- [ ] 自动化测试集：批量执行 + 通过率统计

## 📚 参考学习资料

- [Happy-LLM](https://github.com/datawhalechina/happy-llm) — 大模型原理与实践教程
- OpenAI 兼容 API 文档（Moonshot / DeepSeek / Qwen 等平台通用）

## 📄 License

本项目基于 [MIT License](LICENSE) 开源，仅供学习交流。

---

<div align="center">

**如果这个项目对你有帮助，欢迎点一个 ⭐ Star**

</div>
