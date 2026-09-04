# AI Agent 知识体系

> 目标：建立一套从 LLM（大语言模型）到底层运行时、从单智能体到多智能体、从开发到生产治理的系统学习路径。每个专题都同时回答四类问题：它是什么、为什么这样设计、生产中怎么落地、面试中怎么回答。

配套练习：[AI Agent 高频面试题库](../面试/README.md)

## 0. 全局知识地图

```text
用户目标
  ↓
规则与能力筛选 ──→ 选择模型 / Agent / Workflow（工作流）
  ↓
Context Engineering（上下文工程）
  ├─ Instructions（指令）
  ├─ Conversation（会话）
  ├─ State（运行状态）
  ├─ RAG（检索增强生成）
  ├─ Memory（记忆）
  └─ Tool Definitions（工具定义）
  ↓
Agent Runtime（智能体运行时）
  ├─ LLM 推理与规划
  ├─ Tool Calling（工具调用）
  ├─ 单智能体循环
  ├─ Workflow / DAG（工作流 / 有向无环图）
  └─ Multi-Agent（多智能体）协作
  ↓
外部世界：数据库、搜索、代码执行、业务 API、MCP Server
  ↓
Checkpoint（检查点）/ Trace（轨迹）/ Evaluation（评测）
  ↓
Security（安全）/ Governance（治理）/ Human-in-the-loop（人在回路）
```

一个生产级 Agent 不是“一个写得很长的 Prompt（提示词）”，而是以下部分共同组成的系统：

- LLM 提供理解、推理、生成和不完全可靠的决策能力。
- 上下文工程决定模型当前能看到什么，以及什么内容拥有更高优先级。
- RAG 提供外部事实，Memory 保存跨轮次或跨任务有价值的信息。
- Tool（工具）让模型能读取环境或产生真实副作用。
- Runtime（运行时）负责循环、状态、超时、重试、恢复和终止。
- Workflow 与 Multi-Agent 负责复杂任务的分解、编排与并行。
- Evaluation、Observability（可观测性）和 Security 决定系统能否稳定上线。

## 1. 文档目录

| 专题 | 核心问题 | 建议顺序 |
|---|---|---:|
| [LLM 基础](LLM基础.md) | 模型能做什么、不能保证什么，参数和结构化输出怎样影响 Agent | 1 |
| [上下文工程](上下文工程.md) | 指令、工具、历史、RAG、Memory 如何筛选和组装 | 2 |
| [Agent 与单智能体](Agent与单智能体.md) | Agent 循环、规划、反思、终止和能力路由 | 3 |
| [Tool 与 MCP](Tool与MCP.md) | 工具定义、选择、执行、安全，以及 MCP（模型上下文协议） | 4 |
| [RAG](RAG.md) | 文档解析、检索、重排、上下文组装、幻觉与评测 | 5 |
| [Memory](Memory.md) | 短期/长期记忆，写入、召回、更新、遗忘与污染治理 | 6 |
| [运行时与工作流](运行时与工作流.md) | 状态机、DAG、持久执行、检查点、重试和补偿 | 7 |
| [多智能体](多智能体.md) | 协作模式、任务分解、状态所有权、冲突与容错 | 8 |
| [评测与可观测性](评测与可观测性.md) | 如何衡量结果、过程、工具、成本和线上质量 | 9 |
| [安全与治理](安全与治理.md) | 提示词注入、越权、数据泄露、审批、沙箱和审计 | 10 |

## 2. 必须分清的核心概念

| 概念 | 回答的问题 | 生命周期 | 典型载体 |
|---|---|---|---|
| Prompt（提示词） | 怎样向模型表达当前要求 | 单次调用或模板版本 | system/developer/user message |
| Context（上下文） | 这次推理时模型实际看到了什么 | 单次模型调用 | 完整消息、工具定义、检索内容 |
| State（运行状态） | 当前任务执行到了哪里 | 一次 Run（运行） | 状态对象、数据库记录 |
| Session（会话） | 多轮交互怎样连续 | 一个会话 | 消息历史、会话摘要 |
| Memory（记忆） | 哪些信息值得跨轮次或跨任务复用 | 中长期 | 记忆库、画像、经验库 |
| RAG | 当前问题需要哪些外部事实 | 按查询即时检索 | 文档库、搜索引擎、知识图谱 |
| Checkpoint（检查点） | 失败或暂停后从哪里恢复 | 一次长任务 | 状态快照、待处理动作 |
| Trace（执行轨迹） | 系统实际经历了哪些步骤 | 调试和审计周期 | Span（跨度）、日志、调用链 |
| Cache（缓存） | 哪些确定性或近似结果可以复用 | 按 TTL/版本 | KV、语义缓存、Prompt Cache |

高频面试陷阱：把 Session History（会话历史）直接称为长期记忆；把 Checkpoint 称为 Memory；把 RAG 与 Memory 混成同一个向量库；把 Trace 当业务 State。它们可以使用相同存储技术，但语义、写入策略和生命周期不同。

## 3. 单智能体、多智能体与工作流怎样选

| 方案 | 决策主体 | 适用场景 | 主要风险 |
|---|---|---|---|
| 固定代码 / Workflow | 代码 | 路径清晰、规则稳定、强一致性 | 分支膨胀，难处理开放问题 |
| 单智能体 | 一个 Agent | 目标开放，但工具集合和上下文仍可控 | 循环、误选工具、上下文膨胀 |
| 多智能体 | 多个 Agent + 编排层 | 可独立分解、适合并行或需权限隔离 | 协调成本、冲突、错误传播、成本高 |

推荐决策顺序：

1. 普通代码能可靠解决，就不要引入 LLM 决策。
2. 固定流程能解决，就优先 Workflow。
3. 一个 Agent 加清晰工具能解决，就先做单智能体。
4. 只有存在独立子任务、专业上下文、并行收益或权限隔离需求时，再采用多智能体。

多智能体不是单智能体能力不足时的默认补丁。它主要扩大可并行探索的计算量和上下文容量，同时也引入协调开销。简单任务往往因为通信、重复检索和汇总损失而变差。

## 4. 一次请求的标准处理链路

```text
1. Input Validation（输入校验）
2. Intent / Risk Classification（意图 / 风险分类）
3. Rule Filtering（确定性规则筛选）
4. Capability Routing（能力路由）
5. Context Assembly（上下文组装）
6. Model Decision（模型决策）
7. Tool / Agent / Workflow Execution（执行）
8. Observation（观察执行结果）
9. Continue / Replan / Stop（继续、重规划或终止）
10. Output Validation（输出校验）
11. Memory Write（有条件的记忆写入）
12. Trace / Metrics / Audit（轨迹、指标与审计）
```

其中第 3 步和第 6 步必须分开：

- 确定性规则负责权限、合规、额度、租户隔离、允许调用的工具以及必须审批的动作。
- LLM 适合处理语义意图、模糊分类、开放规划和在允许集合内选择能力。
- 不能把“模型认为可以”当作授权；授权必须由可信代码在执行前重新校验。

## 5. 建议学习路线

### 第一阶段：建立正确模型

学习 LLM、上下文工程、RAG、Tool Calling（工具调用），能解释一次 Agent 请求如何从输入走到工具执行和最终答案。

### 第二阶段：做可靠的单智能体

掌握 Agent Loop（智能体循环）、结构化输出、状态、终止条件、重试、幂等、人工审批和基础 Trace。

### 第三阶段：补齐长期能力

掌握 Memory、Checkpoint、持久化工作流、增量摘要、上下文压缩和长任务恢复。

### 第四阶段：扩展到多智能体

掌握 Manager-as-Tools（管理者调用子智能体）、Handoff（任务移交）、Orchestrator-Workers（编排者—工作者）、并行聚合、冲突解决和状态所有权。

### 第五阶段：生产化

建立离线评测、回归集、线上指标、全链路 Trace、安全边界、审批、审计、成本预算和故障降级。

## 6. 面试回答统一框架

遇到 Agent 系统设计题，可以按以下顺序回答：

1. 先明确业务目标、成功标准和不可接受风险。
2. 判断是否真的需要 Agent，以及为何不用固定工作流。
3. 定义输入、输出、状态和终止条件。
4. 说明上下文由哪些来源组成，如何控制 Token Budget（词元预算）。
5. 说明工具接口、权限、超时、重试、幂等和副作用治理。
6. 说明 RAG 与 Memory 分别保存什么、何时读写。
7. 若用多智能体，说明拆分依据、通信模式、所有权和冲突处理。
8. 说明 Checkpoint、恢复、降级和 Human-in-the-loop。
9. 最后给出离线评测、线上指标、Trace 和安全审计方案。

不要只罗列框架名。面试官更关注：为什么选、失败会怎样、怎样定位、怎样恢复、如何证明改进有效。

## 7. 资料维护原则

- 先讲稳定原理，再介绍具体框架；框架 API 可能变化，核心设计不会快速过时。
- 英文术语首次出现时补充中文解释，后续保留行业常用英文名。
- 任何“效果提升”都需要在业务评测集上验证，不能直接照搬公开案例数字。
- 安全检查必须落在执行边界，不能只依赖 Prompt。
- 文档中的组件边界是逻辑边界，不代表必须部署成独立微服务。

## 参考资料

- [Anthropic：Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic：Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [OpenAI Agents SDK：Agents](https://openai.github.io/openai-agents-python/agents/)
- [OpenAI Agents SDK：Agent Orchestration](https://openai.github.io/openai-agents-python/multi_agent/)
- [Model Context Protocol：Specification](https://modelcontextprotocol.io/specification/)
- [LangGraph：Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
