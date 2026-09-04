# AI Agent 面试题库

> 本题库面向 AI Agent、LLM 应用、RAG 与 Agent 平台工程岗位。题目按照知识域拆分，每题给出可直接复习的标准回答，并通过重点题和系统设计题训练常见追问。

## 1. 题库导航

| 专题 | 主要考点 |
|---|---|
| [01 LLM 基础与推理](01-LLM基础与推理.md) | Transformer、训练与对齐、采样、Embedding、KV Cache、推理优化 |
| [02 Agent 与单智能体](02-Agent与单智能体.md) | Agent 边界、ReAct、规划、路由、循环、上下文和成本 |
| [03 RAG](03-RAG.md) | 解析、分块、混合检索、重排、幻觉、评测、权限和更新 |
| [04 Context 与 Memory](04-Context与Memory.md) | 上下文组装、历史压缩、长短期记忆、写入、召回和污染 |
| [05 Tool 与 MCP](05-Tool与MCP.md) | Function Calling、工具检索、Schema、MCP、授权与可靠性 |
| [06 Runtime 与 Multi-Agent](06-Runtime与Multi-Agent.md) | 状态、检查点、幂等、Saga、多智能体编排、冲突和容错 |
| [07 评测、可观测性与安全](07-评测可观测性与安全.md) | Agent Eval、Trace、线上指标、Prompt Injection、权限与审批 |
| [08 系统设计与项目追问](08-系统设计与项目追问.md) | 端到端设计、容量估算、故障排查、项目深挖和行为面试 |

原入口 [模拟面试题.md](模拟面试题.md) 现在作为兼容导航，不再把全部题目堆在一个文件中。

## 2. 热度标记

- `★★★★★`：几乎所有 Agent / RAG 应用岗位都应掌握。
- `★★★★☆`：中高级岗位经常深入追问。
- `★★★☆☆`：与具体方向相关，掌握后能拉开差距。

热度不是某家公司内部题库泄露，而是依据公开岗位职责中反复出现的能力，以及公开工程资料中常见的生产问题归纳。截至 2026 年，大模型应用、RAG、Agent、AI Coding、AI Safety、长任务执行、工具使用、评测与系统可靠性都是官方岗位描述中的常见主题。

## 3. 推荐复习顺序

### 初级 / 应届

1. LLM 基础与推理
2. Agent 与单智能体
3. RAG
4. Tool 与 MCP
5. Context 与 Memory

目标：概念准确，能解释完整数据流，知道各组件边界。

### 中高级 / 社招

在上述基础上重点学习：

1. Runtime 与 Multi-Agent
2. 评测、可观测性与安全
3. 系统设计与项目追问

目标：能讨论故障、取舍、指标、恢复、安全和实际落地，不只会背框架名。

## 4. 面试回答方法

### 概念题

按“定义 → 原理 → 边界 → 场景 → 风险”回答。先用一句话给结论，再展开，避免一上来罗列名词。

### 系统设计题

按以下顺序：

```text
业务目标与成功标准
  → 请求规模与风险边界
  → 数据流和核心组件
  → State / Context / Tool / RAG / Memory
  → 可靠性与安全
  → 评测、监控、成本
  → 降级与演进路线
```

### 故障排查题

先定义现象和指标，再沿 Trace 找到第一个异常阶段；提出可验证假设和对照实验，最后说明修复如何加入回归集。不要直接回答“换模型”或“调 Prompt”。

### 项目深挖题

使用 STAR（情境、任务、行动、结果）表达，但技术部分还要给出：基线、设计取舍、失败案例、个人贡献和量化结果。无法公开业务数字时，可以说明指标定义和相对变化，不能编造数据。

## 5. 资料来源说明

题目覆盖范围参考公开岗位要求：

- [OpenAI：Software Engineer, Agent Infrastructure](https://openai.com/careers/software-engineer-agent-infrastructure-san-francisco/)
- [OpenAI：Research Engineer, Frontier Evals & Environments](https://openai.com/careers/research-engineer-frontier-evals-and-environments-san-francisco/)
- [字节跳动：前沿技术领域人才校招](https://jobs.bytedance.com/campus/page-6272Gc)
- [华为：AI 大模型训练 / 推理岗位](https://career.huawei.com/reccampportal/portal5/social-recruitment-detail.html?dataSource=1&jobId=28183)
- [华为：Transformer、RAG、Agent 技术栈岗位](https://career.huawei.com/reccampportal/portal5/social-recruitment-detail.html?dataSource=1&jobId=22643)

技术答案优先依据论文与官方文档，并与 [AI Agent 知识体系](../知识体系/README.md) 交叉链接。
