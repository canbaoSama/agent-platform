# Tool 与 MCP 高频面试题

> 返回：[AI Agent 面试题库](README.md)｜配套知识：[Tool 与 MCP](../知识体系/Tool与MCP.md)

## 1. Function Calling 的完整执行过程是什么？ `★★★★★`

### 标准回答

应用向模型提供工具名称、描述和 JSON Schema；模型生成工具名与参数；应用解析后依次做 Schema、业务语义、权限、额度和风险校验；执行工具并处理超时、幂等和错误；再把结构化 Observation 返回模型继续推理或结束。

模型只是生成调用意图，不直接赋予权限。工具执行器必须把模型输出视为不可信外部输入。

## 2. 怎样设计一个模型容易正确调用的工具？ `★★★★★`

### 标准回答

工具应单一职责，名称表达明确动作；描述写明适用、不适用场景和重要限制；Schema 使用清晰字段名、类型、枚举、范围和单位，减少自由文本；结果返回稳定状态、数据、错误码和是否可重试。

避免多个工具功能重叠或要求模型先猜内部 ID。长结果使用分页、摘要或句柄按需读取，不把堆栈和敏感内部字段直接返回模型。用真实任务评测工具选择和参数准确率。

## 3. Structured Output、Function Calling 和 Tool Execution 有什么区别？ `★★★★★`

### 标准回答

Structured Output 约束模型输出结构；Function Calling 是模型用结构化方式表达要调用哪个函数和参数；Tool Execution 是应用在可信环境中真正执行动作。

结构符合 Schema 不代表业务合法，Function Call 也不代表已经执行成功。三层之间必须有校验、授权、错误处理和真实状态验证。

## 4. 上千个工具如何做 Tool Retrieval？ `★★★★★`

### 标准回答

先为工具建立名称、能力、输入输出、领域、权限、风险、环境和版本元数据。检索前按租户、用户权限和风险做确定性过滤；再通过分类、关键词与向量检索召回候选，必要时重排，只将少量完整 Schema 交给主模型。

分类支持多标签和低置信兜底；功能重叠时合并重复工具或明确权威版本、前置条件。评测正确工具 Recall@K、Top-1、参数准确率、误调用、任务成功率、延迟和 Token。

## 5. 如何处理工具错误和重试？ `★★★★★`

### 标准回答

错误至少分为参数校验、权限拒绝、业务冲突、限流/超时、依赖故障和结果未知。网络抖动、临时 5xx 可在 Deadline 内指数退避；参数错误应修正；权限拒绝停止；业务冲突重新读取状态。

写操作超时可能实际已成功，不能直接重试。应使用原幂等键查询结果或重放同一幂等请求。工具返回要明确 `retryable_error` 与 `fatal_error`，避免模型根据自然语言错误猜测。

## 6. MCP 与 Function Calling 有什么区别？ `★★★★★`

### 标准回答

Function Calling 是模型表达工具调用意图的机制。MCP（Model Context Protocol，模型上下文协议）规范 AI Host、Client 和 Server 如何发现并访问 Tools、Resources 和 Prompts，以及连接生命周期和传输。

二者可以组合：Host 通过 MCP 获取工具，再转换成模型的 Function Calling 定义。固定 Workflow 也可直接调用 MCP 工具，因此 MCP 不等于 Agent，也不替代授权、幂等和业务正确性。

## 7. MCP 中 Tools、Resources 和 Prompts 有什么区别？ `★★★★☆`

### 标准回答

- Tools（工具）：模型可选择调用的动作或计算。
- Resources（资源）：应用读取并提供给模型的上下文数据。
- Prompts（提示模板）：通常由用户选择的模板或工作流入口。

它们的控制主体和风险不同。读取 Resource 仍需授权，Tool 尤其要治理副作用；不能因为协议把它描述成 Tool 就默认安全。

## 8. 本地 MCP Server 一定比远程 Server 安全吗？ `★★★★☆`

### 标准回答

不一定。本地 Stdio Server 作为本地进程运行，可能读取文件、环境变量、密钥和网络，供应链或配置错误会直接影响主机。远程 Server 多出网络、OAuth 和服务端信任风险，但更容易做进程隔离和集中更新。

两者都要做来源审查、能力允许列表、最小权限、版本固定、用户确认和调用审计。本地进程还应使用沙箱和受限环境。

## 9. MCP 远程授权有哪些关键点？ `★★★★☆`

### 标准回答

采用 OAuth 相关流程时，Token 应绑定正确 Resource/Audience，使用最小 Scope 和短生命周期；公共客户端使用 PKCE；Server 不接受签发给其他服务的 Token，也不能把上游 Token 直接透传给无关下游。

授权是 Host、Authorization Server 和 Resource Server 的共同责任。每次工具调用仍要做资源级权限校验，连接建立成功不代表可以执行所有能力。

## 10. 如何防止工具越权、危险参数和数据外泄？ `★★★★★`

### 标准回答

模型只看到当前任务允许的工具子集，执行前由服务端基于认证主体、租户、目标资源和真实参数重新授权。参数使用严格 Schema、范围、路径和域名允许列表；高风险动作预览、Dry Run 和人工审批；执行环境限制网络、文件、CPU、时间和凭证。

还要控制数据流向：能读取私有数据的 Agent 不应默认拥有任意外发工具；结果和 Trace 脱敏；所有副作用记录幂等键、审批和真实结果。

## 11. Tool、Skill、Workflow 和 Agent 怎样区分？ `★★★☆☆`

### 标准回答

Tool 是一个边界清楚的动作；Skill（技能包）通常是某类任务的说明、流程、脚本与资源集合；Workflow 用代码编排多个确定步骤；Agent 根据目标和观察动态选择下一步。

Skill 不是所有框架统一的标准协议，常用于 Progressive Disclosure（渐进式披露）：先暴露能力摘要，命中后再加载完整说明。它仍需要版本、权限、安全审查和评测。

## 参考资料

- [Anthropic：Writing Effective Tools for AI Agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [OpenAI Agents SDK：Tools](https://openai.github.io/openai-agents-python/tools/)
- [MCP Specification：Server Features](https://modelcontextprotocol.io/specification/2025-06-18/server/index)
- [MCP Specification：Authorization](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)

