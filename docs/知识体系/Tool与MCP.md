# Tool（工具）与 MCP（模型上下文协议）系统学习指南

> 上位导航：[AI Agent 知识体系](README.md)

> Tool 让模型从“生成文本”扩展为“读取环境、计算和执行动作”。MCP 解决应用如何以统一协议连接外部工具与上下文，不替代工具自身的权限、可靠性和业务设计。

## 0. Tool Calling 的本质

模型通常不直接执行函数，而是生成一个结构化 Tool Call（工具调用请求）：

```text
模型选择工具与参数
  → 应用解析与校验
  → 权限 / 风险 / 审批检查
  → 工具执行
  → 结果规范化
  → 作为 Observation 返回模型
```

工具调用分为两类：

- Read Tool（读工具）：查询、搜索、读取文件，主要风险是越权读取和数据泄露。
- Write Tool（写工具）：发消息、下单、删除、修改配置，具有副作用，需要更强的审批、幂等和审计。

“只读”也不等于安全。读取私密数据后，模型仍可能通过答案或另一个工具泄露信息。

## 1. 工具契约设计

一个好工具要让模型容易正确选择，也让运行时容易验证。

### 1.1 名称与描述

- 名称表达一个明确动作，例如 `search_orders`、`refund_order`。
- 描述说明何时使用、何时不使用、返回什么以及重要限制。
- 避免多个工具名称相似、边界重叠。
- 把业务概念写清，不依赖模型猜内部缩写。

### 1.2 Input Schema（输入模式）

使用 JSON Schema 明确：

- 必填与可选字段；
- 类型、枚举、格式与范围；
- 字段语义和单位；
- 互斥或条件关系；
- 稳定资源 ID，而非模糊自然语言名称。

```json
{
  "name": "refund_order",
  "description": "对已支付且符合政策的订单发起退款；高于限额时需要审批",
  "parameters": {
    "type": "object",
    "properties": {
      "order_id": {"type": "string", "description": "订单唯一标识"},
      "amount_cents": {"type": "integer", "minimum": 1},
      "reason": {"type": "string"},
      "idempotency_key": {"type": "string"}
    },
    "required": ["order_id", "amount_cents", "reason", "idempotency_key"],
    "additionalProperties": false
  }
}
```

Schema 只能保证结构，不能保证订单属于当前用户或退款金额合法，这些必须在服务端校验。

### 1.3 Output Contract（输出契约）

工具结果建议统一为：

```json
{
  "status": "success | partial | retryable_error | fatal_error | denied",
  "data": {},
  "error": {"code": null, "message": null},
  "metadata": {"request_id": "...", "has_more": false}
}
```

返回给模型的信息要面向下一步决策：状态明确、字段稳定、错误可分类、结果可分页。内部堆栈、密钥和无关大字段不要进入模型上下文。

## 2. 工具选择与工具检索

工具很少时可全部传入；工具很多时应做 Tool Retrieval（工具检索）：

1. 按租户、身份、环境和风险过滤；
2. 按领域或意图粗分；
3. 用关键词、Embedding（嵌入）或分类器召回候选；
4. 用规则或模型重排；
5. 只把少量相关工具完整 Schema 放入上下文。

需要评测：正确工具是否进入候选集、模型是否选中、参数是否正确、是否产生不必要调用。只评最终答案无法定位是检索错、选择错还是执行错。

## 3. 执行层可靠性

### 3.1 错误分类

- Validation Error（校验错误）：修正参数，不盲目重试。
- Authorization Error（授权错误）：停止或请求额外授权。
- Business Conflict（业务冲突）：读取新状态后重新规划。
- Rate Limit / Timeout（限流 / 超时）：在预算内退避重试。
- Dependency Failure（依赖失败）：熔断、降级或排队。
- Unknown Outcome（结果未知）：先查询幂等记录或真实状态，不能直接重放写操作。

### 3.2 幂等与重复调用

每个副作用动作使用业务幂等键，并由服务端记录请求和最终结果。模型生成的自然语言理由不能代替幂等标识。

### 3.3 超时与取消

向下传播 Deadline（截止时间）和 Cancellation（取消信号）。用户取消或上层超时后，下层工具不能继续悄悄产生副作用。

### 3.4 补偿与人工处理

多步骤副作用使用 Saga（长事务）记录已完成步骤及补偿动作。不可逆或高价值动作必须在执行前审批，而不是事后“补偿”。

## 4. 工具安全边界

执行前至少检查：

- 当前主体和租户是否有权调用；
- 目标资源是否在授权范围；
- 参数是否满足业务和合规规则；
- 是否需要 Human Approval（人工审批）；
- 是否超过频率、金额、Token 或时间预算；
- 调用是否携带最小权限凭证；
- 外部内容是否可能触发 Prompt Injection（提示词注入）。

高风险工具应使用 Sandbox（沙箱）、网络和文件系统隔离、域名允许列表、临时凭证、资源配额与完整审计。

## 5. MCP 是什么

MCP 是连接 AI 应用与外部能力的开放协议。其架构包含：

- Host（宿主）：面向用户的 AI 应用，负责整体安全和生命周期。
- Client（客户端）：宿主中与某一个 MCP Server 保持连接的协议组件。
- Server（服务器）：通过标准协议暴露能力和上下文。

核心 Server Primitives（服务器原语）：

- Tools（工具）：模型可发现并调用的动作。
- Resources（资源）：应用可读取的上下文数据。
- Prompts（提示模板）：用户可选择的模板或工作流入口。

协议层常使用 JSON-RPC（JSON 远程过程调用），标准传输包括 Stdio（标准输入输出）与 Streamable HTTP（可流式 HTTP）。具体实现应以目标版本规范为准。

## 6. MCP 不解决什么

MCP 统一的是发现、描述和调用接口，但不会自动解决：

- 工具业务逻辑是否正确；
- 用户是否有权操作具体资源；
- 写操作是否幂等；
- Prompt Injection 是否会诱导工具误用；
- 工具返回是否可信、是否过大；
- 多租户隔离、审计、审批与灾难恢复。

MCP Server 不能因为“连接成功”就被视为可信。Host 仍需做 Server Allowlist、能力审查、用户授权和每次调用校验。

## 7. MCP Authorization（授权）

远程 MCP 的授权建立在 OAuth（开放授权）相关机制上。工程重点：

- Token（令牌）要绑定正确 Resource / Audience（资源 / 受众）；
- 使用最小 Scope（作用域）和短生命周期；
- 禁止 Token Passthrough（令牌透传）到不相关下游；
- 公共客户端使用 PKCE（代码交换证明密钥）；
- Server 不能接受本来签发给其他服务的令牌；
- Host 要明确告知用户将访问什么数据、执行什么动作。

本地 Stdio Server 也需要信任管理，因为它作为本地进程运行，可能访问文件、环境变量和网络。

## 8. MCP 与 Function Calling 的关系

| 维度 | Function Calling | MCP |
|---|---|---|
| 关注点 | 模型怎样输出工具名和参数 | 应用怎样发现、连接和调用外部能力 |
| 范围 | 单个模型 API / SDK 的调用机制 | Client-Server 协议和生命周期 |
| 工具来源 | 应用代码直接注册 | MCP Server 动态暴露 |
| 是否可组合 | 可以，MCP Tool 常映射成模型工具 | 依赖宿主把能力提供给模型 |

两者不是竞争关系：Host 可通过 MCP 获取工具列表，再转换为模型支持的 Tool Calling 格式。

### 8.1 Tool、Skill、Workflow 与 Agent 不要混淆

| 概念 | 主要作用 | 典型形态 |
|---|---|---|
| Tool | 执行一个边界清楚的能力 | 函数、API、MCP Tool |
| Skill（技能包） | 向 Agent 渐进式提供某类任务的说明、流程和配套资源 | 指令文件、脚本、模板、参考资料 |
| Workflow | 用确定性结构编排多个步骤 | 代码、状态机、DAG |
| Agent | 根据目标和观察动态决定下一步 | 模型 + 上下文 + 工具 + 运行时 |

Skill 不是所有框架都统一定义的协议概念，但它代表一种常见工程方法：平时只向模型暴露能力名称和简述，命中相关任务后再加载完整说明与资源。它能减少基础 Prompt 和工具描述长度，但仍需版本、权限、安全审查与效果评测。

## 9. 常见失败与排错

| 现象 | 检查项 | 可能修复 |
|---|---|---|
| 从不调用正确工具 | 候选集、描述、Schema、示例 | 改边界、做工具检索、减少重叠 |
| 参数经常错误 | 字段语义、单位、枚举、条件关系 | 强化 Schema、先解析实体、服务端返回可修正错误 |
| 重复下单/发消息 | 超时、重试、幂等记录 | 幂等键、状态查询、副作用检查点 |
| MCP 能连接但调用失败 | 协议版本、能力协商、认证、超时 | 记录握手与请求 ID，区分协议和业务错误 |
| 工具结果撑爆上下文 | 分页、裁剪、结构化摘要 | 返回引用/句柄，按需取详情 |
| 越权读取 | 主体、租户、资源级校验 | 服务端授权，不能信模型传入的 user_id |

## 10. 面试重点

### 10.1 Tool Calling 的完整流程是什么？

答题要点：模型根据工具定义生成名称和参数；应用解析并做结构、语义、权限、风险校验；执行端处理超时、幂等和重试；结果结构化返回模型；最终验证真实环境状态并记录 Trace。

### 10.2 工具数量很多时怎么办？

答题要点：先按权限和规则过滤，再用领域分类或语义检索召回，重排后只暴露少量完整 Schema。分别评估候选召回率、工具选择率和参数准确率。

### 10.3 MCP 和 Function Calling 有什么区别？

答题要点：Function Calling 是模型输出结构化调用意图的机制；MCP 是 Host、Client、Server 之间发现和访问 Tools、Resources、Prompts 的协议。MCP 工具最终仍可通过 Function Calling 被模型选择。

### 10.4 怎样防止 Prompt Injection 导致工具误用？

答题要点：外部内容标记为不可信；最小化工具与权限；执行前由策略引擎校验真实参数；高风险动作审批；沙箱与网络隔离；监控异常调用链。Prompt 提醒只是辅助。

### 10.5 工具调用超时后可以直接重试吗？

答题要点：读操作通常可在预算内重试；写操作可能已成功但响应丢失，应先通过幂等键或查询接口确认状态，再决定返回已有结果、重试或补偿。

## 11. 术语速查

| 英文 | 中文 | 含义 |
|---|---|---|
| Tool Calling | 工具调用 | 模型选择工具并生成参数的机制 |
| Tool Retrieval | 工具检索 | 从大量能力中召回少量候选工具 |
| JSON Schema | JSON 模式 | 描述参数结构、类型和约束 |
| MCP | 模型上下文协议 | AI 应用连接工具和上下文的开放协议 |
| Host / Client / Server | 宿主 / 客户端 / 服务器 | MCP 的三个核心角色 |
| Resource | 资源 | MCP 暴露的上下文数据 |
| OAuth | 开放授权 | 委托访问授权框架 |
| PKCE | 代码交换证明密钥 | 防止授权码被截获滥用的机制 |

## 参考资料

- [Anthropic：Writing Effective Tools for AI Agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [OpenAI Agents SDK：Tools](https://openai.github.io/openai-agents-python/tools/)
- [OpenAI Agents SDK：Guardrails](https://openai.github.io/openai-agents-python/guardrails/)
- [MCP Specification：Server Features](https://modelcontextprotocol.io/specification/2025-06-18/server/index)
- [MCP Specification：Transports](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [MCP Specification：Authorization](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
