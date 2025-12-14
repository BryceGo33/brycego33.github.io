---
title:      Context Engineering
date:       2025-08-17
categories:
  - Tech
tags:
  - Agent
  - Langchain
---

最近看到langchain blog中，提及了一个新概念，叫Context Engineering，乍一眼看着像是Prompt Engineering Plus？其实还挺不一样的...

> Context engineering is the delicate art and science of filling the context window with **just the right** information **for the next step**.（上下文工程是一门精妙的艺术和科学，旨在在上下文窗口中填入**恰到好处**的信息，为下一步推理做准备）- Andrej Karpathy（这是个绝对的大神！！！吴恩达老师的同事）
>  The art of providing all the context for the task to be plausibly solvable by the LLM.（为LLM提供所有合理的任务背景，使任务能够被LLM合理地解决。）- Tobi Lütke（Shopify CEO）

### What
![优化路径](/assets/images/2025-08-17%20Context%20Engineering/file-20250816214458955.png)
1. **LLM Optimization**（约束解码、FT等）
都是高成本的事情啊
话说LLM的SFT年初搞了一波，OpenAI还是牛逼的，把SFT整得这叫一个方便（hg好好学学人家）...但是参数就开放了那么几个，有点难受，估计也是怕被猜出他们用的模型架构？
然后hg新搞的那个什么可视化的训练平台AutoTrain，就纯纯一辣鸡玩意儿，一点儿都不好用，功能乱七八糟不说，还一堆bug，复现效果复现半个礼拜，结果还差了20个点😡纯一好看的玩具。
2. 所以**Context Optimization**才是王道啊

![Context Engineering](/assets/images/2025-08-17%20Context%20Engineering/file-20250816214458955%201.png)

1. **指导性上下文**：这类上下文的核心功能是**指导模型该做什么以及如何做**。主要为模型行为设定框架，目标和规则。它包括：
	- Prompt Chain
	- System prompts（Task Description, etc.）
	- Dynamic elements of the prompt (e.g., user inputs, date/time, etc.)
	- Few-shot Examples
	- Output Schema

2. **信息性上下文**，这类上下文的核心功能是**告诉模型需要知道什么知识**。为模型提供解决问题所必备的事实，数据与知识。它包括：
	- RAG
	- **Store(Long-term Memory)**
	- **State(Short-term Memory)**
	- Scratchpad（Todo-list, Thinking, etc.）
	- **Runtime(Variable)**
	
3. **行动性上下文**，这类上下文的核心功能是**告诉模型能做什么，以及做了之后的结果**。为模型提供与外部世界交互的能力。它包括：
	- Tool Definition
	- Tool Calls
	- Tool Results

### Why

如果不做context engineering，会导致什么？

| 原因               | 导致                      |
| ---------------- | ----------------------- |
| 上下文缺失            | 准确率下降                   |
| 上下文干扰（噪声、冲突、混乱等） | 准确率下降，成本激增              |
| 上下文溢出            | 成本激增，时延激增、甚至timeout阻塞流程 |
| 未保护隐私&安全         | 内部信息泄露                  |

### How：
Context Engineering常见的四大策略：
![](/assets/images/2025-08-17%20Context%20Engineering/file-20250816214458955%202.png)

其他策略：
1. 命中更多的KV-Cache -> 降本提速
![](/assets/images/2025-08-17%20Context%20Engineering/file-20250817214458955%203.png)
2. Context Metadata
    例如：context本身加matadata（timestamp / source / reliability等），后续根据这些 metadata 判断可信度 /优先级 /是否过期 等
3. RAG相关：动态RAG、分层RAG、图RAG等等
4. 强制引用
等等（要学的还好多啊啊啊啊啊）

### Final
感觉公司项目目前啥都有点，但啥都好像没有做精...网上的方案也都是五花八门的，或是因为大家也都还在找一个统一的范式？
看越多，不会的也越来越多了。抓紧再看点东西吧骚年，应该还是有点用的。
上个月调研的supervisor agent，下月似乎就能落地了，这估计是我头一次完全一人撑起的项目？有点激动也有点忐忑...可别出啥幺蛾子啊。


### Ref
1. https://www.promptingguide.ai/guides/context-engineering-guide
2. https://www.promptingguide.ai/agents/context-engineering
3. https://platform.openai.com/docs/guides/optimizing-llm-accuracy
4. https://blog.langchain.com/context-engineering-for-agents/
5. https://manus.im/zh-cn/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
6. https://mp.weixin.qq.com/s/EcPPM8JoaYnabd4PgB3AjA
7. https://zhuanlan.zhihu.com/p/1938967453951571269?share_code=AIpuKNtPFO8z&utm_psn=1977698052790834567
