# AI Agent 工程师技术学习手册（2026 版）

---

> 更新日期：2026-08-17  
> 适用目标：AI Agent 开发工程师、LLM 应用工程师、AI 平台工程师  
> 目标：系统掌握生产级 Agent 的设计、实现、评测、安全与平台化能力。  

> 本手册专注技术学习、实践路线与项目建设；公司面经、专项题库和求职冲刺内容已拆分到《AI Agent 工程师大厂面试手册》。

---

## 0. 先看结论

三个岗位不是三套完全不同的知识，而是同一技术栈的三个深度层级：

| 岗位                | 核心问题                               | 面试官最关心的证据                               |
| ------------------- | -------------------------------------- | ------------------------------------------------ |
| LLM 应用工程师      | 模型怎样解决具体业务问题               | RAG/结构化输出、质量、成本、业务指标             |
| AI Agent 开发工程师 | 模型怎样可靠地使用工具完成多步任务     | Agent Loop、状态、工具、Memory、评测、安全       |
| AI 平台工程师       | 怎样让很多团队安全稳定地运行很多 Agent | Runtime、多租户、分布式系统、沙箱、可观测性、SRE |

合理的学习顺序是：

```text
Python/后端基础
    ↓
LLM 原理与 API 工程
    ↓
Prompt + Structured Output + Tool Calling
    ↓
RAG + Evaluation
    ↓
Agent Loop + Workflow + Memory + MCP
    ↓
安全、Trace、成本、可靠性
    ↓
分布式 Runtime、Sandbox、多租户与平台工程
```

不要一开始就做复杂 Multi-Agent。先把一个 Agent 的正确率、可观测性、失败恢复和权限边界做好。

---

---

## 1. 岗位能力地图

### 1.1 三类岗位共同必修

- Python、异步编程、类型系统、测试；
- HTTP、REST、SSE、WebSocket、鉴权和数据库；
- Transformer 基础、Token、Context Window、Embedding；
- Prompt、结构化输出、Function Calling；
- RAG、向量检索、混合检索、Rerank；
- Agent Loop、状态、工具和错误恢复；
- Evaluation、Trace、日志、延迟和成本分析；
- Prompt Injection、越权工具调用、敏感数据保护；
- Docker、Redis、PostgreSQL 和基础云部署。

### 1.2 不同岗位的深度要求

| 技术              | LLM 应用 | Agent 开发 |    AI 平台 |
| ----------------- | -------: | ---------: | ---------: |
| Prompt/结构化输出 |       深 |         深 |         中 |
| RAG/检索质量      |       深 |         深 |     中到深 |
| Agent 状态机      |       中 |         深 |         深 |
| MCP/工具协议      |       中 |         深 |         深 |
| Evaluation        |       深 |         深 |         深 |
| FastAPI/业务后端  |       深 |         深 |         深 |
| 分布式系统        |       中 |     中到深 |       极深 |
| Kubernetes/调度   |     基础 |         中 |         深 |
| Sandbox/虚拟化    |     了解 |         中 |         深 |
| 多租户/IAM/审计   |       中 |         深 |       极深 |
| GPU 训练/推理底层 |     了解 |       了解 | 视岗位而定 |

OpenAI 当前公开的 Agent Infrastructure、Codex Core Agents 和 Cloud Agents 职位把生产级 Agent 平台能力描述得很具体：长任务编排、沙箱与隔离、状态和记忆、可靠性、可观测性、安全、Token/延迟/成本优化，以及 Python/Rust/分布式系统和云基础设施。这可以作为平台岗位能力上限的参照，而不是要求初学者一次全部掌握。

---

---

## 2. 软件工程基本功

### 2.1 Python

#### 原理与必会点

- 数据类型、迭代器、生成器、装饰器和上下文管理器；
- `typing`、`Protocol`、泛型、Pydantic 数据校验；
- `asyncio` 事件循环、协程、Task、并发限制和取消；
- 异常体系、重试边界、资源释放；
- 包管理、虚拟环境、日志、配置和测试。

LLM 应用大量时间花在网络 I/O：同时调用模型、检索服务和业务 API。异步的价值不是“让单次调用更快”，而是等待 I/O 时让同一进程继续处理其他工作。

#### 当前主流设计

- FastAPI + Pydantic 构建类型化 API；
- `async/await` 贯穿 HTTP、数据库和模型调用；
- 领域逻辑与模型 SDK 解耦，通过 `Protocol`/Adapter 接入供应商；
- `pytest` + Fake Model/Fake Tool 做确定性测试；
- Ruff 做 lint/format，`pyright` 或 `mypy` 做类型检查。

#### 重点难点

- 把阻塞 SDK 放进异步函数，导致整个事件循环被阻塞；
- 无限制 `gather`，瞬间打满模型限额或数据库连接池；
- 吞异常后返回“模型失败”，失去根因；
- 重试一个有副作用的 Tool，造成重复扣款、重复发信；
- 把框架对象传遍业务层，后续无法替换框架。

#### 快速上手练习

实现一个并发模型网关：

1. 定义 `ModelProvider` 接口；
2. 实现两个 Fake Provider；
3. 加超时、限流、重试和熔断；
4. 记录 Token、延迟和 request ID；
5. 写测试验证超时、取消和降级。

#### 高频面试题

- `asyncio` 并发和多线程/多进程有什么区别？
- 如何限制同时调用模型的请求数？
- 客户端取消 SSE 后，后端怎样停止模型和工具调用？
- 什么错误可以重试？怎样保证幂等？

### 2.2 API、数据库和缓存

#### 必会知识

- HTTP 状态码、Header、Cookie/JWT/OIDC；
- REST 资源建模与幂等键；
- SSE 单向流、WebSocket 双向通信；
- PostgreSQL 索引、事务、隔离级别和连接池；
- Redis 缓存、分布式锁、Stream/PubSub 的适用边界；
- 对象存储、预签名 URL 和文件生命周期。

#### 典型设计

```text
POST /runs -> 返回 run_id
GET  /runs/{id}/events -> SSE 流
POST /runs/{id}/cancel -> 取消
POST /approvals/{id}/approve -> 从 Checkpoint 恢复
```

#### 难点

- SSE 断线重连后的事件去重和游标续传；
- 数据库状态与消息队列投递的一致性，可使用 Outbox Pattern；
- 缓存击穿、雪崩和过期数据；
- 多租户查询漏加 `tenant_id` 导致横向越权；
- 长事务占用连接，拖垮整个服务。

---

---

## 3. LLM 基础原理

### 3.1 Transformer 需要理解到什么程度

应用岗位无需从头训练大模型，但要能解释：

- Tokenization：文本被切成模型处理的离散 Token；
- Embedding：Token/文本被表示为高维向量；
- Self-Attention：每个位置根据 Query、Key、Value 聚合上下文；
- 位置编码：让模型知道序列顺序；
- 预训练与指令微调：从“预测下一个 Token”到“遵循任务指令”；
- 推理模型：会使用额外推理计算解决复杂任务；
- Context Window：一次请求可见的输入、工具结果和历史上限。

Attention 的简化表达：

```text
Attention(Q, K, V) = softmax(QKᵀ / √d) V
```

面试重点不是背公式，而是把原理连接到工程：

- 上下文越长，延迟和成本通常越高；
- 把无关文档塞进 Prompt 会稀释有效信号；
- 模型输出是概率生成，不是数据库确定性查询；
- Temperature 影响采样多样性，不等于“事实正确率旋钮”；
- 相同模型在不同任务上的最佳推理强度不同，必须通过 Eval 选择。

### 3.2 模型分类与选择

#### 分类

- 通用模型：问答、写作、提取、工具调用；
- 推理模型：复杂规划、数学、编码与多步判断；
- 小模型：分类、路由、抽取和高并发低成本任务；
- Embedding 模型：语义检索、聚类和去重；
- Reranker：对候选文档重新排序；
- 多模态模型：文本、图像、音频或视频；
- 开源/开放权重模型：私有部署、定制和数据边界场景。

#### 当前主流设计

不是“全系统只用最强模型”，而是模型路由：

```text
简单分类/抽取 -> 小模型
普通生成/工具选择 -> 平衡型模型
复杂规划/最终审查 -> 强推理模型
敏感或本地数据 -> 私有模型
失败或限流 -> 降级模型
```

OpenAI 当前官方建议在推理、工具调用和多轮工作流中采用 Responses API，并强调要用代表性评测比较质量、延迟和成本，而不是默认把推理强度开到最高。具体模型名称和价格变化很快，项目中应保存“模型能力标签与价格快照”，不要把业务逻辑写死在模型名上。

#### 重点难点

- 质量、速度、成本不能同时最优；
- 不同供应商的 Tool Calling、流式事件和错误语义不同；
- 模型升级可能改变格式、工具偏好和回答长度；
- 供应商限流分请求数与 Token 数，且可能按模型分别计算；
- 模型评测数据若不代表真实流量，路由结论会失真。

#### 面试题

- 如何为客服、报告生成和代码 Agent 选择模型？
- 为什么不能只看公开 Benchmark？
- 模型升级怎样做灰度、回滚和兼容性验证？
- 怎样设计多供应商 Model Gateway？

---

---

## 4. Prompt 与 Context Engineering

### 4.1 Prompt 的组成

```text
System/Developer Policy
+ Task Goal
+ Business Context
+ Available Tools
+ Retrieved Knowledge
+ Conversation/State
+ Output Schema
+ Examples
```

Prompt Engineering 关注怎样写指令；Context Engineering 关注运行时“选择什么信息、何时加入、以什么优先级加入、何时压缩或移除”。

### 4.2 当前主流设计

- 明确目标、约束、权限边界和成功标准；
- 使用结构化输出而非解析自然语言；
- Tool 描述短而准确，只暴露当前任务相关工具；
- 动态检索上下文，不把所有资料永久塞入系统提示；
- 长会话使用摘要、状态字段和按需回取；
- Prompt、工具 Schema、模型和评测集一起版本化；
- 每次修改 Prompt 都跑回归 Eval。

OpenAI 当前模型指导强调精简重复指令、只提供相关工具、明确自主执行与审批边界，并通过已有评测验证 Token 减少是否保持质量。

### 4.3 难点

- 指令冲突：系统规则、用户要求、文档内容相互矛盾；
- Lost in the Middle：关键信息淹没在长上下文中；
- Prompt Injection：文档或网页伪装成指令；
- Few-shot 示例过多造成成本和过拟合；
- 摘要不断压缩后丢失事实和约束；
- Tool Schema 太宽，模型容易填错或越权。

### 4.4 面试回答框架

被问“如何优化 Prompt”时，不要只回答“多试几次”：

1. 定义可测指标；
2. 收集真实失败样本；
3. 给失败分类：知识缺失、指令不清、工具失败、模型能力不足；
4. 每次只改变一个变量；
5. 在固定数据集回归；
6. 比较质量、Token、延迟和成本；
7. 灰度上线并监控。

---

---

## 5. Structured Output 与 Tool Calling

### 5.1 原理

模型不直接执行函数。它生成一个符合工具 Schema 的调用意图，应用程序负责：

```text
模型选择工具并生成参数
        ↓
应用校验 Schema 与权限
        ↓
应用实际执行工具
        ↓
结果作为 Tool Result 返回模型
        ↓
模型继续判断或生成最终答案
```

### 5.2 工具分类

- 只读工具：搜索、查库存、查数据库；
- 有副作用工具：发邮件、创建工单、更新记录；
- 破坏性工具：删除、转账、生产变更；
- 本地工具：文件、Shell、浏览器；
- 远程工具：HTTP API、MCP Server；
- 确定性工具：计算器、规则引擎；
- 非确定性工具：搜索、另一个 Agent。

### 5.3 当前主流设计

- JSON Schema 严格输入；
- Tool Allowlist：每一步只暴露最小工具集；
- Tool Risk Level：read-only / side-effect / destructive；
- Policy Enforcement：服务端校验，不依赖 Prompt；
- Human-in-the-loop：高风险动作暂停审批；
- Idempotency Key：防止重试产生重复副作用；
- Tool Result 截断、摘要和可信标记；
- 并行调用只用于互不依赖且安全的工具；
- 有界循环：最大步骤、最大成本、停止条件。

### 5.4 难点

- 模型选对工具但参数不完整；
- 工具返回错误时模型反复重试；
- 并行工具之间存在隐藏依赖；
- 工具返回的数据包含恶意指令；
- 审批等待数小时后，原始数据已经变化；
- 工具调用成功但响应丢失，系统不知道是否该重试。

### 5.5 必做练习

实现一个“报销审批 Agent”：

- `query_policy`：只读；
- `calculate_amount`：确定性；
- `create_reimbursement`：有副作用且幂等；
- 超过阈值触发人工审批；
- 任一步骤均生成 Trace；
- 用 Fake Tools 测试批准、拒绝、超时和重复提交。

---

---

## 6. RAG：检索增强生成

### 6.1 原理

RAG 不是“向量库 + Prompt”这么简单。完整链路：

```text
离线：
文件 -> 解析 -> 清洗 -> Chunk -> Metadata -> Embedding -> Index

在线：
问题 -> 改写/分解 -> 权限过滤 -> 召回 -> 融合 -> Rerank
     -> Context Packing -> LLM -> 引用 -> 反馈/Eval
```

### 6.2 分类

#### 按召回方式

- Sparse Search：BM25/关键词，擅长专有名词和精确匹配；
- Dense Search：Embedding 向量，擅长语义相似；
- Hybrid Search：稀疏与向量融合；
- Graph Retrieval：基于实体关系和图遍历；
- SQL/Metadata Retrieval：结构化过滤和聚合。

#### 按控制方式

- Naive RAG：固定检索一次；
- Advanced RAG：改写、混合检索、Rerank、压缩；
- Agentic RAG：Agent 判断是否检索、检索什么、是否继续；
- Corrective RAG：先判断召回质量，必要时改写或换数据源；
- Multi-hop RAG：分解问题，多轮检索合成。

### 6.3 当前主流生产设计

优先把以下组合做好：

```text
版面感知解析
+ 语义/结构混合 Chunk
+ Metadata/ACL
+ Hybrid Search
+ Rerank
+ Token-aware Context Packing
+ 可点击引用
+ Retrieval Eval
```

GraphRAG 和 Agentic RAG 很热门，但不是所有知识库都需要。企业手册、制度问答通常先把解析、权限、混合检索和 Rerank 做好，收益比直接上知识图谱更确定。

### 6.4 核心难点

#### 文档解析

- PDF 阅读顺序、页眉页脚、表格和扫描件；
- Excel 的行列语义；
- 图片、公式和跨页表格；
- 文档版本和重复内容。

#### Chunk

- 太小：缺上下文；
- 太大：召回不准、Token 贵；
- 固定长度切断标题与段落关系；
- 同一策略不适合合同、FAQ、代码和表格。

#### 检索

- Query 与文档表达方式不同；
- 相似但无答案的内容排名靠前；
- 权限过滤放在召回后导致数据泄露；
- Rerank 增加延迟和成本；
- 只评最终回答，不知道问题出在召回还是生成。

### 6.5 必会指标

- Recall@K：包含答案的片段是否被召回；
- Precision@K：召回内容中相关内容比例；
- MRR/NDCG：正确结果排序质量；
- Context Relevance：送给模型的上下文是否相关；
- Faithfulness/Groundedness：回答是否忠于上下文；
- Citation Correctness：引用是否真的支持结论；
- Answer Correctness：最终答案是否正确。

### 6.6 面试题

- Chunk Size 怎样选？为什么不能给出一个通用值？
- Hybrid Search 为什么通常优于单一向量检索？
- 如何保证不同部门的文档不会互相召回？
- RAG 回答错了，怎样定位是解析、召回、排序还是生成问题？
- 什么情况下应该用 GraphRAG？

---

---

## 7. Agent 原理与架构

### 7.1 Agent 是什么

一个最小 Agent 是“模型在循环中读取状态、决定下一步并调用工具”：

```text
Observe -> Decide -> Act -> Observe -> ... -> Finish
```

Agent 相比普通 Workflow 的区别：

- Workflow 的路径主要由代码预先决定；
- Agent 的部分路径由模型根据当前状态动态决定。

生产系统通常是二者混合：用确定性 Workflow 控制主流程，让模型只在需要语义判断的节点决策。

### 7.2 常见架构

#### ReAct

模型在推理与行动之间循环。容易上手，但长循环可能成本高、难以稳定。

#### Plan-and-Execute

先生成计划，再逐步执行。适合长任务，但计划可能很快过时，需要重规划。

#### Router

模型只负责分类和分流，后续走确定性子流程。可靠、便宜，适合企业流程。

#### Supervisor / Multi-Agent

Supervisor 分配任务给多个专用 Agent。适合可清晰拆分的任务，但带来通信、重复工作、上下文和成本问题。

#### Agent Harness

为通用 Agent 提供计划、文件系统、Shell、子 Agent、技能、上下文管理和安全边界，Coding Agent 是典型场景。

### 7.3 当前主流设计

当前更成熟的方向不是“无限自治”，而是：

- 有限状态图；
- Durable Execution 与 Checkpoint；
- 关键动作 Human-in-the-loop；
- 动态工具选择与最小权限；
- 长任务可暂停、恢复和取消；
- Context Compaction；
- Trace + Eval + Feedback Loop；
- Sandbox 中执行不可信代码；
- 单 Agent 优先，确有并行收益时再 Multi-Agent。

LangGraph 官方将 durable execution、streaming、human-in-the-loop 和 persistence 作为核心能力；Checkpoint 使运行可恢复、可回放、可分支，也是长期任务和审批流程的基础。

### 7.4 状态设计

好的 State 保存事实，不保存已经格式化的 Prompt：

```python
class AgentState(TypedDict):
    goal: str
    plan: list[PlanStep]
    observations: list[Observation]
    artifacts: list[ArtifactRef]
    budget: Budget
    status: RunStatus
```

难点：

- State 过大导致序列化和恢复昂贵；
- 多节点并行更新同一字段产生冲突；
- Checkpoint 在副作用动作之前还是之后；
- 恢复时外部世界已经变化；
- 框架升级后旧状态能否反序列化。

### 7.5 错误分类

```text
瞬时错误 -> 有界重试 + 退避
模型可修复错误 -> 把结构化错误返回模型
用户可修复错误 -> 暂停并请求输入
权限/安全错误 -> 立即拒绝并审计
未知错误 -> 失败退出，保存 Trace/Checkpoint
```

### 7.6 高频面试题

- Agent 与 Workflow 有什么区别？何时不用 Agent？
- 怎样避免 Agent 无限循环？
- Tool 执行成功但进程崩溃，恢复后怎样避免重复执行？
- 长任务怎样暂停、恢复、取消？
- 为什么 Multi-Agent 不一定优于单 Agent？
- 如何设计 Agent State 和 Checkpoint？

---

---

## 8. LangGraph、OpenAI Agents SDK 与其他框架

### 8.1 框架分类

- 高层 Agent Framework：快速创建常见 Agent Loop；
- 低层 Orchestration Runtime：显式状态、图、持久化和恢复；
- Agent SDK：工具、Handoff、Guardrail、Trace 等标准能力；
- Durable Workflow Engine：Temporal 等通用长任务引擎；
- Agent Harness：Coding/Research Agent 的完整运行外壳。

### 8.2 选择原则

| 需求                                   | 建议                              |
| -------------------------------------- | --------------------------------- |
| 简单问答 + 少量工具                    | 直接模型 SDK 或高层 Agent SDK     |
| 明确状态图、审批、分支、恢复           | LangGraph                         |
| 跨天长任务、严格 Exactly-once 业务活动 | Agent Runtime + Temporal 类引擎   |
| 多模型快速试验                         | 保持自有接口，框架放适配器层      |
| Coding Agent                           | Harness + Sandbox + 文件/终端工具 |

### 8.3 面试不要只背 API

面试官更关注：

- 为什么选这个框架；
- 框架在持久化、重试和恢复时做了什么；
- 怎样替换框架；
- 哪些状态属于你的业务，哪些属于框架；
- 如果不用框架，最小 Runtime 怎样实现。

---

---

## 9. MCP

### 9.1 原理

MCP 是 Host、Client、Server 架构。数据层基于 JSON-RPC，传输层常见：

- `stdio`：本地进程通信；
- Streamable HTTP：远程连接，可配合流式传输。

Server 可暴露三类核心原语：

- Tools：可执行动作；
- Resources：可读取上下文；
- Prompts：可复用交互模板。

连接初始化时进行协议版本与能力协商，然后客户端动态发现 Server 能力。

### 9.2 MCP 与 Function Calling 的区别

```text
Function Calling：
模型供应商定义“模型如何表达工具调用”

MCP：
定义 AI 应用如何发现、连接并调用外部上下文/工具服务
```

两者经常组合：MCP Client 发现工具，转换成模型可理解的 Tool Schema；模型产生调用，Host 再通过 MCP 执行。

### 9.3 当前重点

- 远程 MCP 的 OAuth、身份传递和租户隔离；
- Tool/Resource 动态发现和版本兼容；
- Streamable HTTP；
- 权限与用户确认；
- Server 健康、限流、超时和审计；
- 工具结果内容的可信度边界。

### 9.4 难点

- Server 声明的工具并不代表当前用户有权限；
- 动态 Tool Schema 变化导致 Prompt/Eval 漂移；
- 本地 Server 可能拥有过大的文件或 Shell 权限；
- 远程 Server 面临 SSRF、Token 转发和会话固定风险；
- 多个 Server 工具重名或语义相近；
- 将几百个工具全部提供给模型会降低选择准确率并增加 Token。

### 9.5 必做练习

实现一个最小 MCP Server：

- Resource：项目 README；
- Tool：搜索项目文件；
- Prompt：代码审查模板；
- 同时用 stdio 与 Streamable HTTP 测试；
- 客户端做工具白名单、超时和审计；
- 为危险 Tool 增加审批。

---

---

## 10. Memory

### 10.1 分类

- Working Memory：当前一次运行的状态；
- Short-term Memory：当前 Thread 的消息和摘要；
- Long-term Semantic Memory：用户偏好、业务事实；
- Episodic Memory：过去任务和结果；
- Procedural Memory：可复用流程、规则或技能；
- Knowledge Base：组织文档，不应与用户记忆混为一谈。

### 10.2 当前主流设计

- Thread Checkpoint 与跨 Thread Store 分离；
- Memory 写入经过抽取、验证、去重和保留策略；
- Namespace 至少包含租户和用户；
- 共享规则默认只读；
- 敏感记忆写入需要明确政策或人工确认；
- 按需检索 Memory，不把全部记忆注入每轮上下文；
- 支持查看、更正、过期和删除。

### 10.3 难点

- 把模型推测当成用户事实永久保存；
- 多用户共享 Memory 导致隐私泄露或 Prompt Injection；
- 新旧事实冲突；
- 记忆越积越多，召回噪声不断上升；
- 删除关系数据库记录却漏删向量/缓存；
- 摘要记忆不可追溯到原始来源。

### 10.4 面试题

- 对话历史和长期 Memory 有什么区别？
- 什么时候写入 Memory？谁有权删除？
- 怎样处理“用户换工作了”这样的事实更新？
- 怎样防止一名用户向共享记忆注入恶意指令？

---

---

## 11. Evaluation：最容易拉开候选人差距的能力

### 11.1 为什么必须评测

LLM 系统具有非确定性，Prompt、模型、RAG、工具或数据任何一项变化都可能导致回归。没有 Eval，团队只能凭 Demo 感觉判断。

### 11.2 分类

- 离线 Eval：固定数据集，适合开发和发布门禁；
- 在线 Eval：真实流量、反馈、人工抽检；
- 组件 Eval：检索、分类、工具选择、结构化输出；
- 端到端 Eval：最终任务是否完成；
- 确定性评分：Exact Match、规则、程序验证；
- 人工评分：专家依据 Rubric 判断；
- LLM-as-a-Judge：规模化语义评分；
- Pairwise Eval：候选版本与基线两两比较；
- Red Team Eval：越权、注入、敏感数据和危险行为。

### 11.3 当前主流设计

```text
生产失败样本
   ↓
清洗与去敏
   ↓
加入分层测试集
   ↓
候选版本 vs 基线
   ↓
质量 + 安全 + 延迟 + 成本
   ↓
发布门禁与灰度监控
```

数据集要按任务类型、难度、用户群和风险分层。LLM Judge 必须用人工标注样本校准，不能成为唯一发布依据。

### 11.4 指标

- 任务完成率；
- 工具选择/参数正确率；
- 检索 Recall@K 和引用正确率；
- 结构化输出 Schema 通过率；
- Hallucination/Groundedness；
- Prompt Injection 防御通过率；
- P50/P95 延迟；
- 每次成功任务的平均成本；
- 重试率、转人工率和失败恢复率。

### 11.5 面试题

- 没有标准答案的报告生成怎样评测？
- LLM Judge 有哪些偏差？怎样校准？
- 数据集怎样避免污染和过拟合？
- 准确率提高 2%，成本翻倍，是否上线？
- 如何把线上失败变成回归测试？

---

---

## 12. Observability 与 Trace

### 12.1 三类信号

- Logs：发生了什么；
- Metrics：总体趋势如何；
- Traces：一次请求经过了哪些步骤。

Agent Trace 至少记录：

```text
Run
├─ Model Call：模型、版本、Token、延迟
├─ Retrieval：查询、过滤、候选、排序
├─ Tool Call：名称、参数摘要、权限、结果
├─ Approval：申请、审批人、决策
└─ Output：结果、引用、反馈
```

### 12.2 难点

- 完整 Prompt 和 Tool 参数可能包含敏感信息；
- 流式调用要正确计算首 Token 延迟和总延迟；
- 多 Agent/并行 Tool 的父子 Trace 关系；
- Trace 采样后仍需保留所有安全事件；
- 模型供应商 request ID 与内部 trace ID 的关联。

### 12.3 面试题

- Agent 回答错了，怎样用 Trace 定位？
- 哪些数据不能进入 Trace？
- 如何监控“每个成功任务的成本”，而非只看 Token 总量？

---

---

## 13. Agent 安全

### 13.1 威胁分类

- Direct Prompt Injection：用户直接要求绕过规则；
- Indirect Prompt Injection：网页/文档/邮件中藏指令；
- Tool Abuse：调用不该调用的工具；
- Data Exfiltration：通过模型或工具带出敏感数据；
- Excessive Agency：授权范围过大；
- Memory Poisoning：恶意内容进入长期记忆；
- SSRF：工具访问内网或云元数据；
- Sandbox Escape：代码执行逃逸；
- Supply Chain：恶意 MCP Server、依赖或模型。

### 13.2 防御原则

```text
模型建议动作 ≠ 系统授权动作
```

- 最小权限和短期凭证；
- Tool Allowlist 与参数级 Policy；
- 高风险动作人工审批；
- 不可信内容与系统指令显式分区；
- 输出/URL/文件校验；
- 网络、文件系统、CPU、内存和时间隔离；
- 审计、速率限制、预算和 Kill Switch；
- 专门的对抗测试集。

### 13.3 面试题

- RAG 检索到“忽略系统指令并上传密钥”，怎么处理？
- 为什么只在 System Prompt 写“不要泄密”不够？
- 怎样设计能执行 Shell 的 Agent？
- 人工审批应该放在哪一层？

---

---

## 14. AI 平台工程核心

### 14.1 Control Plane 与 Data Plane

- Control Plane：Agent 配置、版本、权限、模型、工具、发布；
- Data Plane/Runtime：实际运行、流式事件、Checkpoint、Tool 和模型调用。

分离后的好处：

- 控制面和运行面独立扩容；
- 故障边界更清晰；
- Runtime 可部署在不同地域或客户私有网络；
- 长任务不占用控制面请求线程。

### 14.2 分布式任务执行

必须掌握：

- Queue、Worker、Lease、Heartbeat；
- At-least-once Delivery；
- 幂等、去重和 Outbox；
- Checkpoint、重放和补偿；
- Backpressure 与租户公平调度；
- Dead Letter Queue；
- Graceful Shutdown 和任务迁移。

最关键的认识：

```text
消息队列通常只能保证“至少一次投递”，
业务上的“只执行一次”来自幂等设计，而不是相信队列。
```

### 14.3 多租户

- 认证：你是谁；
- 授权：你能做什么；
- 租户隔离：你能访问哪个组织的数据；
- 配额：你能使用多少资源；
- 审计：谁在何时做了什么。

隔离层：

- API/Policy；
- PostgreSQL Row/Schema/Database；
- Redis Key Namespace；
- Qdrant Payload Filter/Collection；
- Object Storage Prefix/Bucket；
- Runtime Network/Credential。

### 14.4 Sandbox

从弱到强：

- 进程级限制；
- 容器；
- gVisor/Kata；
- MicroVM（如 Firecracker）；
- 独立主机/集群。

隔离越强，启动时间、资源成本和运维复杂度通常越高。平台岗位需要能根据威胁模型选择，而不是只回答“用 Docker”。

OpenAI 当前公开平台岗位明确提及 Kata、Firecracker、gVisor、Sysbox、容器编排和大规模分布式基础设施，说明对高自治、可执行代码的 Agent，沙箱已经是平台核心能力。

### 14.5 SRE 与成本

- SLI：成功率、延迟、可恢复率、队列等待；
- SLO：例如月度 Run 成功率和 P95 首 Token；
- Error Budget；
- 熔断、降级、限流和容量规划；
- Token、模型、工具、存储、网络的成本归因；
- 按 Tenant/Agent/Version 计量；
- 模型升级和供应商故障演练。

### 14.6 平台岗面试题

- 设计一个支持十万长任务 Agent 的 Runtime；
- Worker 执行工具后崩溃，如何恢复？
- 怎样保证租户公平，防止大客户饿死小客户？
- 如何隔离可以运行不可信代码的 Agent？
- SSE 网关如何支持断线续传和水平扩展？
- PostgreSQL、Redis、Qdrant 各存什么，为什么？
- 怎样做多模型限流、成本核算和故障降级？

---

---

## 15. Multi-Agent

### 15.1 适用场景

- 任务可以清晰拆成相互独立的研究或执行流；
- 不同子任务需要不同工具、权限或上下文；
- 并行执行能显著降低墙钟时间；
- 每个子任务都有可验证输出；
- Supervisor 能在有限预算内合并结果。

### 15.2 不适用场景

- 本来一个确定性 Workflow 就能完成；
- 子任务高度依赖、需要频繁沟通；
- 没有 Eval 判断协作是否真的更好；
- 只是为了在简历上写 Multi-Agent；
- 成本、延迟和故障点不可接受。

### 15.3 难点

- 任务分解质量；
- 重复搜索和重复 Tool 调用；
- 子 Agent 结论冲突；
- 上下文在 Agent 间复制导致成本膨胀；
- Supervisor 成为单点瓶颈；
- 权限继承和审计链；
- 并发任务的取消与收敛。

面试时应主动说明 Multi-Agent 的成本，并给出单 Agent/Workflow 基线对照实验。

---

---

## 16. 前端能力

前端不是平台的装饰层。必须处理：

- SSE Token 与步骤事件；
- 运行状态机；
- 取消、重试、审批和恢复；
- Trace 时间线与树；
- Prompt/Agent 版本 Diff；
- 知识引用定位；
- Workflow 图编辑；
- 大量事件的虚拟列表；
- 敏感参数遮罩；
- 网络断线和重复事件。

Vue 3 面试重点：

- Composition API 与响应式原理；
- TypeScript Props/Events；
- Pinia/状态建模；
- Router 权限；
- SSE/WebSocket 生命周期；
- 大列表性能；
- 可测试组件与 API 契约。

---

---

## 17. 如何快速上手：12 周路线

### 第 1-2 周：后端与模型 API

- FastAPI、Pydantic、asyncio、pytest；
- 流式模型调用；
- 结构化输出；
- 超时、重试、限流、Token 记录。

交付物：支持两个 Provider 的 Model Gateway。

### 第 3-4 周：RAG

- 文档解析、Chunk、Embedding；
- Qdrant；
- Hybrid Search + Rerank；
- 引用；
- Recall@K 与回答评测。

交付物：企业知识库问答，附 50 条评测集。

### 第 5-6 周：Agent 与工具

- ReAct、Router、Plan-and-Execute；
- LangGraph State/Node/Edge；
- Tool Calling；
- Checkpoint、重试、取消。

交付物：能调用三个业务工具的单 Agent。

### 第 7 周：MCP 与 Memory

- MCP Server/Client；
- stdio/Streamable HTTP；
- 短期与长期 Memory；
- Namespace 与权限。

交付物：一个 MCP Server + 可删除的用户记忆。

### 第 8 周：Eval 与 Observability

- Trace；
- 组件/端到端 Eval；
- LLM Judge 校准；
- Token、成本、P95 延迟。

交付物：候选版本与基线的评测报告。

### 第 9 周：安全与审批

- Prompt Injection；
- Tool Policy；
- Human-in-the-loop；
- 幂等和审计。

交付物：高风险工具必须审批，恶意测试集不可绕过。

### 第 10 周：异步 Runtime

- Queue、Worker、Lease、Heartbeat；
- Outbox、DLQ；
- Checkpoint 恢复。

交付物：API 进程重启后任务仍可恢复。

### 第 11 周：平台化

- Tenant、RBAC、AgentVersion、Deployment；
- 配额、模型路由、密钥引用；
- Docker Compose。

交付物：两个租户之间无法访问对方资源。

### 第 12 周：面试包装

- 架构图；
- README 和一键启动；
- 20 个真实失败案例；
- 性能与成本数据；
- 3 分钟、10 分钟、30 分钟三版项目讲解；
- 模拟系统设计与故障排查。

---

---

## 18. 三个递进项目

### 项目 A：企业知识库 Agent

证明 LLM 应用能力：

- PDF/DOCX 解析；
- Hybrid Search + Rerank；
- 权限过滤与引用；
- RAG Eval；
- SSE；
- 成本和延迟看板。

面试必须能给出数据：

```text
Recall@5 从多少提升到多少
引用正确率多少
P95 延迟多少
每次成功回答成本多少
最常见的三类失败是什么
```

### 项目 B：任务规划 Agent

证明 Agent 工程能力：

- LangGraph；
- 任务理解、规划、执行、复盘；
- MCP/HTTP Tools；
- Checkpoint；
- Human Approval；
- 预算、重试、取消和 Trace；
- 单 Agent 与 Multi-Agent 对比。

### 项目 C：企业级 Agent Platform

证明平台能力：

- Agent/Version/Deployment；
- Model Gateway；
- Tool Registry；
- Runtime/Worker；
- Tenant/RBAC/Audit；
- Eval/Trace；
- Sandbox 设计；
- 配额、成本和 SLO。

不必一次做全。先实现一条端到端纵切：

```text
创建 Agent
-> 发布版本
-> 发起 Run
-> 流式查看步骤
-> 工具审批
-> 完成
-> 查看 Trace/Eval
```

---

---

## 19. 学习取舍

### 优先深入

- Python 异步后端；
- RAG 质量与 Eval；
- Tool Calling、Agent State、Checkpoint；
- LangGraph；
- MCP；
- Trace、成本和安全；
- PostgreSQL、Redis、Docker；
- 一个可量化的完整项目。

### 掌握概念和基本实践

- Kubernetes；
- Temporal 类 Durable Workflow；
- OpenTelemetry；
- GraphRAG；
- Multi-Agent；
- 开源模型部署；
- gRPC。

### 目标不是算法岗时暂缓深入

- 从零训练大模型；
- CUDA Kernel；
- 大规模分布式训练；
- RLHF 算法细节；
- 为追热点同时学习十个 Agent 框架。

---

---

## 20. 官方资料与持续更新

以下资料用于核对本文的当前技术方向：

- [OpenAI 模型与 Agent 工作流指导](https://developers.openai.com/api/docs/guides/latest-model)
- [OpenAI 当前模型目录](https://developers.openai.com/api/docs/models)
- [OpenAI Software Engineer, Agent Infrastructure](https://openai.com/careers/software-engineer-agent-infrastructure-san-francisco/)
- [OpenAI Software Engineer, Codex Core Agents](https://openai.com/careers/software-engineer-codex-core-agents-san-francisco/)
- [OpenAI Software Engineer, Cloud Agents](https://openai.com/careers/software-engineer-cloud-agents-san-francisco/)
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [LangGraph Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- [LangGraph Human-in-the-loop](https://docs.langchain.com/oss/python/langchain/human-in-the-loop)
- [LangGraph Memory](https://docs.langchain.com/oss/python/langgraph/add-memory)
- [Model Context Protocol Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [MCP Specification Architecture](https://modelcontextprotocol.io/specification/2025-06-18/architecture)
- [Anthropic MCP Overview](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)

框架、模型名称和职位要求会快速变化。每月更新具体产品和 API，每季度重新检查能力地图；稳定不变的主线仍然是软件工程、检索、工具、状态、评测、安全与可靠性。

---

---

## 21. 对现有能力体系的审查结论

### 23.1 总体评价

原文的技术主线是正确的，而且完整度已经高于多数只覆盖 Prompt、RAG、LangChain 的 Agent 学习资料。它已经覆盖生产级 Agent 最重要的八项能力：

1. 后端与异步编程；
2. LLM API、Prompt 和结构化输出；
3. RAG 与检索评测；
4. Agent Loop、状态、工具和恢复；
5. MCP 与 Memory；
6. Evaluation、Trace 和成本；
7. 安全、审批和权限；
8. 分布式 Runtime 与多租户平台。

如果目标是普通 AI 应用/Agent 开发岗，原文主体已经覆盖约 85% 的知识面；如果目标是腾讯、字节、阿里等大厂中高级岗位，还需要补齐以下能力。

### 23.2 必须补充的能力

| 补充方向 | 为什么重要 | 掌握深度 |
| --- | --- | --- |
| Agent Skill 与动态能力路由 | 2026 年 Agent 越来越强调按需加载 Prompt、工具和知识，避免单一超长 Prompt | 必须能设计并实现 |
| Coding Agent / AI Coding | 大厂岗位开始直接要求熟悉 CLI、Skills、Agent 和 AI Coding 工具链 | 至少完成一个可演示项目 |
| 多模态与 GUI/Browser Agent | 手机、桌面、浏览器和企业流程 Agent 经常需要图像理解与界面操作 | 掌握架构、评测和安全 |
| 推理服务基础 | 腾讯部分岗位直接要求 vLLM、TensorRT-LLM、分布式推理或模型服务经验 | 应用岗理解，平台岗实操 |
| 微调与对齐边界 | 面试常问什么时候用 Prompt、RAG、SFT、LoRA、DPO，而不一定要求亲自训练大模型 | 能做选型和实验设计 |
| 计算机基础与算法 | 腾讯面经中经常把 Agent 与网络、并发、数据库、Linux、算法题混合考察 | 必须补强 |
| 数据闭环与产品指标 | 真正上线的 Agent 必须说明用户价值、失败样本、反馈闭环和量化收益 | 社招尤其重要 |

### 23.3 不建议继续扩张的内容

- 不要同时学习十个 Agent 框架，主线保持“原生模型 SDK + LangGraph/PydanticAI + MCP”即可；
- 不要为了面试堆砌 Multi-Agent，先证明单 Agent/Workflow 基线为什么不够；
- 不要把 CUDA、分布式训练、RLHF 全部列为 Agent 应用岗必修；
- 不要只背概念，每个核心模块必须有代码、测试、Trace 和量化结果；
- 不要把框架的示例项目直接写进简历，面试官会通过追问快速识别。

---

---

## 22. Agent Skill、路由与按需上下文

### 24.1 Skill 的定位

Skill 是某类任务可复用的执行说明、领域规则和资源集合，不等同于 Agent，也不等同于输出格式。

```text
基础 Prompt：全局身份、公共约束、安全边界
Skill：某类任务的分析方法、步骤和专属知识
Tool：查询数据或执行确定性动作
Output Schema：约束最终返回字段
Renderer：把结构化结果渲染成地图、卡片或 Markdown
Router：决定本次加载哪些 Skill、Tool 和 Schema
```

### 24.2 推荐架构

```text
请求
  -> 显式 request_type
  -> 规则/小模型路由兜底
  -> 加载 Skill 元数据
  -> 加载命中的完整 Skill
  -> 动态注册最小工具集
  -> 单 Agent 执行
  -> Schema 校验
  -> Renderer 展示
```

路由优先级：

1. 业务系统直接传 `request_type`，确定性最高；
2. 使用规则匹配明确意图；
3. 无法判断时使用小模型结构化分类；
4. 低置信度时追问用户，不要静默猜测。

### 24.3 Skill 设计原则

- 描述中明确“做什么”和“什么情况下触发”；
- Skill 正文只保留流程、约束和资源导航；
- 详细业务规则放入按需读取的 references；
- 重复、确定性高的操作写成 scripts；
- 模板、图标和固定文件放入 assets；
- 一次请求可以组合多个 Skill，但最终应选择一个输出 Schema；
- Skill 不宜拆得过细，应该围绕独立业务目标划分。

### 24.4 高频面试题

1. Skill、Tool、Prompt、Workflow 和 Agent 有什么区别？
2. 如何避免所有 Skill 都进入上下文？
3. Skill 路由错误怎么发现和评测？
4. 一次请求命中多个 Skill 时如何处理冲突？
5. Skill 升级怎样做版本管理、灰度和回滚？
6. 为什么输出 JSON 不是一个业务 Skill？
7. Tool 数量达到上千时怎样检索、过滤和授权？
8. 怎样防止恶意 Skill 或工具说明注入系统指令？

---

---

## 23. Coding Agent、多模态与推理服务补充

### 25.1 Coding Agent

需要理解的核心组件：

- 仓库扫描、符号检索、全文检索和依赖关系；
- Plan、Todo、文件编辑、Patch 和终端执行；
- 测试、Lint、类型检查和失败修复循环；
- Git Diff、变更边界和用户已有修改保护；
- Shell 权限、网络权限、Secret 保护和 Sandbox；
- 长上下文压缩、任务交接和可恢复执行；
- Skill/规则文件的发现与按需加载。

建议项目：实现一个“小型代码修复 Agent”，要求接收 Issue、定位文件、生成 Patch、运行测试、失败后最多修复两轮，并输出修改摘要和风险说明。

### 25.2 多模态和 GUI Agent

至少能解释：

- 截图/图片如何进入模型；
- OCR、视觉定位与语义理解怎样组合；
- GUI 操作使用坐标、DOM、Accessibility Tree 还是混合方式；
- 页面变化后如何验证动作是否成功；
- 如何处理弹窗、登录、验证码和异步加载；
- 如何构建任务成功率、步骤准确率和安全评测；
- 为什么高风险点击必须审批或使用确定性 API 替代。

### 25.3 推理服务基础

应用岗至少掌握：

- Prefill 与 Decode；
- KV Cache 的作用和显存占用；
- Continuous Batching；
- TTFT、TPOT、吞吐和 P95 延迟；
- Quantization 的收益与质量风险；
- vLLM/PagedAttention 的基本思想；
- 模型路由、限流、排队、熔断和降级。

平台岗继续深入 Tensor Parallel、Pipeline Parallel、Speculative Decoding、张量并行通信、GPU 调度和容量规划。

### 25.4 Prompt、RAG、微调的选型

| 问题 | 优先方案 |
| --- | --- |
| 指令不清、格式不稳定 | Prompt + Structured Output |
| 外部知识经常变化且需要引用 | RAG |
| 固定风格、分类边界或工具习惯长期不稳定 | SFT/LoRA 候选 |
| 需要偏好对齐 | DPO/RL 类方法候选 |
| 要求模型记住实时业务数据 | 不应依赖微调，使用 RAG/Tool |

面试中不要直接回答“微调效果更好”。应该先定义数据、基线、成本和评测，再通过受控实验决定。

---
