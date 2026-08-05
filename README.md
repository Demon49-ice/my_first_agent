# 我的第一个手写 Agent

学习大模型应用开发期间的练手项目，不依赖任何 Agent 框架，
从零实现 Agent Loop 和工具调用。

## 文件说明
- `多轮对话.py`：多轮对话程序（system 设定 / 记忆 / Token 统计 / 滑动窗口）
- `agent.py`：手写 Agent（工具调用 / Agent Loop / 异常兜底）

## 实现的功能
- 模型自主决定是否调用工具（计算器、话费查询）
- 多轮记忆与上下文管理
- 工具异常兜底与死循环防护

## 运行方法
1. pip install openai
2. 填入 Moonshot API Key
3. python agent.py

## 效果截图
（这里贴一两张运行截图）
