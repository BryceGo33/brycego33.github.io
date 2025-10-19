---
title:      supervisor-agent
date:       2025-07-11
categories:
  - Tech
tags:
  - Agent
  - Langchain
---

这应该是我第一次写高度结合工作上用到的技术的博客，尝试一下。

## 简介

随着多智能体（Multi-Agent）系统的发展，如何在复杂任务中实现**高效的智能体协作与调度**成为关键问题。

在真实业务中，模型之间往往存在角色分工：有的负责信息提取，有的负责逻辑推理，有的负责生成回复。

如果所有智能体都平级协作，虽然灵活，但容易造成上下文混乱与任务重复；而若完全由一个模型负责所有任务，又难以兼顾精度与可扩展性。

为了解决这一矛盾，于是引入了 **Supervisor Agent** —— 一种结合了单智能体与多智能体优势的中枢式调度架构。

本文将介绍 Supervisor Agent 的概念、架构实现、在业务中的验证结果，以及其与 Swarm 模式的区别与优势。

---

## 概念与原理

### 两大多智能体流派

在多智能体系统中，主流架构可分为两类：

| 类型 | 特点 | 优势 | 局限 |
|------|------|------|------|
| **Supervisor Agent** | 由一个中心智能体调度若干子智能体 | 结构清晰、易于管控 | 灵活性略低 |
| **Swarm Agent** | 各智能体自治、并行协作 | 去中心化、鲁棒性高 | 上下文管理复杂 |

*Supervisor Agent*
![](/assets/images/supervisor-agent/supervisor-agent.png)

*Swarm Agent*
![](/assets/images/supervisor-agent/swarm-agent.png)

### Supervisor Agent 的混合特性

Supervisor Agent 架构兼具了 **Single-Agent** 和 **Multi-Agent** 的双重特性：

1. **【Single 模式】** 用户仅与 Supervisor 交互，整体体验统一；
2. **【Multi 模式】** Supervisor 能根据任务动态切换至不同子 Agent 完成具体子任务。

这种结构既保持了多智能体的**分工与专业性**，又保留了单智能体系统的**一致性与可控性**。

---

## 架构与交互机制

Supervisor Agent 的核心在于 **“中心调度 + 显式迁移”**。
它通过 LangGraph 的节点与边机制，实现了多智能体之间的灵活切换与任务流转。

*supervisor路由示意图*
![](/assets/images/supervisor-agent/supervisor-router.png)

---

### 一、内部跳转逻辑

在 LangGraph 框架中，Supervisor 与子 Agent 之间的跳转是通过内置的 **Command** 实现的。

**1. Supervisor → 子 Agent**

- 自动添加 `transfer_to_xxx_agent` 工具；
- 模型根据上下文判断触发时机；
- 跳转逻辑通过 `Command`+`goto` 实现。

**2. 子 Agent → Supervisor**

- 自动添加无条件边（`add_edge`）；
- 子 Agent 执行完成后自动回流至 Supervisor；
- 逻辑在图结构层由 `add_edge` 内部封装。

这种“显式goto + 自动transfer back”模式保证了系统流向清晰、可调试，且在复杂多轮任务中保持稳定的上下文衔接。

---

### 二、外部交互方式

在我设计的Demo代码中，Supervisor 被设计为“无状态”的。
它的工作方式与单智能体类似：每次通过接口调用与用户或工具交互，而不保留历史状态。

这种架构带来以下优点：

- **可扩展性强**：每次调用独立，不依赖持久化上下文；
- **轻量化**：不需要在内部实现tool的调用等功能

---

### 三、关键参数解析

Supervisor 架构中有两个关键参数，用于控制上下文可见性与迁移信息展示：
- `output_mode`
  - `full_history`：子 Agent 能看到完整上下文（智能度更高，但计算成本更大）；
  - `last_message`：仅传递上一条消息（轻量、默认模式）。
- `add_handoff_messages`
  控制是否在日志或界面中显示迁移提示，如"Successfully transferred to extraction_agent"，便于调试与追踪。

---

## 实践效果与验证
在前期实验中，我们主要验证了以下方面：
	1.	功能正确性：Supervisor 能稳定完成多 Agent 的任务调度；
	2.	上下文一致性：在多轮切换中保持语境连续；
	3.	扩展性：可轻松接入新的子 Agent（如检索、推理、生成、校验等模块）；
	4.	性能表现：相较 Swarm 架构，Supervisor 模式在响应延迟与出错率上更优。

在“智能问答 + 工具调用”场景下，Supervisor 自动判断任务分派逻辑，使冗余调用减少约 30%。

---

### 未来方向
Supervisor Agent 的潜力远不止于任务分配，我们正探索以下方向：
	1. 层级化 Supervisor：支持多层中枢结构，处理更复杂的分布式任务；
	2. 添加memory机制：让 Supervisor 拥有跨任务、跨会话的上下文记忆；
	3. 作为voice agent的基础框架，处理这种相对意图多变的场景。

---

### 参考文档

1. [Supervisor Github项目](https://github.com/langchain-ai/langgraph-supervisor-py)
    
2. [Langgraph supervisor文档](https://langchain-ai.github.io/langgraph/agents/multi-agent/#supervisor)

3. 对于multi agent system和context engineering的态度，两种主流思想代表：

- 【偏single】[Cognition AI](https://cognition.ai/blog/dont-build-multi-agents#a-theory-of-building-long-running-agents)
        
- 【偏multi】[Anthropic](https://www.anthropic.com/engineering/built-multi-agent-research-system)
