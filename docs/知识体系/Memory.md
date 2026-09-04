# Memory（记忆）系统学习指南

> 上位导航：[AI Agent 知识体系](README.md)

> Agent Memory 的核心不是“把所有对话存下来”，而是让系统在合适的时间，以可控、可验证、可删除的方式复用过去信息。写入质量通常比存储容量更重要。

## 0. Memory 解决什么问题

LLM 单次调用只知道当前上下文。Memory 让 Agent 在跨轮次、跨会话或跨任务时复用：

- 用户稳定偏好和已确认事实；
- 当前长期任务的进展与决定；
- 过去事件和交互经历；
- 被验证有效的操作经验或程序；
- 对后续任务有价值的失败教训。

Memory 不应存：未经确认的模型猜测、短期无关闲聊、密钥、超出用途的敏感信息，以及无法追溯来源的摘要。

## 1. Memory 分类

### 1.1 按持续时间

- Working Memory（工作记忆）：当前 Run 内的临时信息，通常属于 Context 或 State。
- Short-term Memory（短期记忆）：当前会话或近期任务需要的信息。
- Long-term Memory（长期记忆）：跨会话仍有价值的信息。

### 1.2 按认知功能

- Semantic Memory（语义记忆）：稳定事实与概念，如“用户默认使用中文”。
- Episodic Memory（情景记忆）：带时间和上下文的事件，如“上周部署因配额失败”。
- Procedural Memory（程序性记忆）：怎样做事的经验、策略或步骤。
- Preference Memory（偏好记忆）：用户明确或高置信推断的偏好。

分类不是为了模仿人脑，而是因为不同记忆需要不同的写入、召回、冲突和过期策略。

## 2. Memory、RAG、Session 与 State 的区别

| 对象 | 主要内容 | 典型读取方式 | 典型写入方 |
|---|---|---|---|
| RAG | 组织文档、知识库、外部事实 | 查询时检索 | 文档管道或知识维护者 |
| Memory | 从交互与任务中沉淀的信息 | 按主体、任务、语义、时间召回 | Agent + 受控写入策略 |
| Session | 某次会话的消息连续性 | 按会话 ID 读取 | 会话运行时 |
| State | 当前 Run 的确定性状态 | 运行时直接读取 | 节点、工具和编排器 |

Memory 与 RAG 都可能使用向量数据库，但不能因此混为一谈。RAG 强调知识来源、索引和权限；Memory 更强调主体、时间、置信度、冲突、遗忘和用户控制。

## 3. Memory 生命周期

```text
事件 / 对话 / 工具结果
  → Candidate Extraction（候选提取）
  → Write Decision（写入决策）
  → Normalize & Link Source（规范化与关联来源）
  → Store（存储）
  → Retrieve（召回）
  → Rank & Filter（排序与过滤）
  → Use in Context（进入上下文）
  → Update / Consolidate / Forget（更新 / 合并 / 遗忘）
```

每一步都需要策略，不能只做“每轮对话向量化”。

## 4. 写入机制

### 4.1 什么值得写

可以按以下维度评分：

- Future Utility（未来效用）：后续是否可能用到；
- Stability（稳定性）：信息是否短期就会变化；
- Confidence（置信度）：用户明确确认还是模型推断；
- Novelty（新颖度）：是否已有相同记忆；
- Sensitivity（敏感度）：是否允许保存；
- Scope（范围）：个人、团队、租户还是单任务。

### 4.2 同步写入与异步写入

- 同步写入：立即可用，适合用户明确要求“记住”；但增加延迟。
- 异步写入：在后台提取、去重和合并；吞吐更好，但下一轮可能暂不可见。

高风险或用户可见的记忆建议支持确认、查看、修改和删除。

### 4.3 推荐字段

```json
{
  "memory_id": "mem_...",
  "subject_id": "user_...",
  "type": "preference",
  "content": "回答技术问题时优先使用中文",
  "source_refs": ["message_123"],
  "confidence": 1.0,
  "valid_from": "2026-09-03T08:00:00Z",
  "valid_to": null,
  "scope": "user",
  "sensitivity": "normal",
  "version": 1
}
```

来源、有效期、范围和版本与内容同样重要。

## 5. 召回与排序

只按向量相似度容易召回“话题相似但已经过期”的内容。推荐综合：

```text
score = semantic_relevance
      + importance
      + recency_decay
      + confidence
      + scope_match
      - contradiction_penalty
      - sensitivity_risk
```

先做主体和权限过滤，再做语义召回；不能先跨租户向量搜索后再过滤，因为检索结果、缓存和日志都可能泄露信息。

### 5.1 召回结果进入上下文前

- 合并重复记忆；
- 检查有效期和来源；
- 发现冲突时优先明确确认、更新且权威的记录；
- 限制每种记忆的数量和 Token；
- 标明“明确事实”还是“低置信推断”；
- 对敏感记忆再次做用途与权限校验。

## 6. 更新、冲突与遗忘

### 6.1 Append-only（仅追加）的问题

只追加会保留大量互相矛盾的旧偏好。需要 Supersede（替代）关系或版本链：新记录生效，旧记录保留审计但不再召回。

### 6.2 冲突策略

优先级通常是：用户当前明确陈述 > 较新的明确陈述 > 高置信系统事实 > 模型推断。无法判断时不要静默覆盖，应询问用户或保留“不确定”。

### 6.3 Forgetting（遗忘）

遗忘不是缺陷，而是质量机制：

- TTL（生存时间）到期；
- 使用频率长期为零；
- 已被新版本替代；
- 用户请求删除；
- 不再满足保存目的或合规要求；
- 低价值记忆合并成更高层摘要。

删除需覆盖主存储、向量索引、缓存和派生副本，并保留合规允许的最小删除审计。

## 7. Memory Consolidation（记忆整合）

情景记忆积累后，可以抽取稳定模式形成语义或程序性记忆。例如，多次观察到用户偏好简洁回答，只有达到阈值或经用户确认后才形成长期偏好。

整合风险：模型可能把偶然行为错误概括为稳定偏好。应保留支持事件、置信度和可撤销关系。

## 8. Memory Pollution（记忆污染）

常见来源：

- 模型把自己的猜测当事实写回；
- 恶意用户或外部文档诱导写入错误指令；
- 工具失败信息被总结成成功经验；
- 不同用户、租户或任务的记忆混用；
- 过期信息持续影响决策。

治理措施：

- 写入 Allowlist 与敏感字段过滤；
- 用户陈述、工具事实、模型推断分级；
- 关键事实要求工具或用户确认；
- 写入和读取均做租户隔离；
- 对程序性记忆进行离线评测后再发布；
- 保存来源并支持回滚、删除和冲突分析。

## 9. Memory 架构选型

### 9.1 存储层

- Relational Database（关系数据库）：结构化字段、版本、事务与权限。
- Vector Store（向量存储）：语义召回非结构化记忆。
- Search Index（搜索索引）：关键词、过滤和混合搜索。
- Object Store（对象存储）：原始事件、长文本和附件。
- Graph Store（图存储）：实体关系、事件链和复杂关联。

常见做法是关系库保存真相与元数据，向量索引做候选召回，而不是把向量库当唯一真相源。

### 9.2 分层内存

MemGPT（现常以分层记忆思想被引用）提出类似操作系统虚拟内存的思路：把有限上下文作为“主存”，把外部存储作为“辅存”，由系统在需要时换入换出。工程价值在于明确区分当前可见上下文与外部长期记忆。

## 10. Memory 评测

- Write Precision（写入精确率）：写入的内容有多少真正值得长期保存。
- Write Recall（写入召回率）：应保存的信息是否被保存。
- Retrieval Recall / Precision（召回率 / 精确率）。
- Conflict Resolution Accuracy（冲突解决准确率）。
- Staleness Rate（过期记忆命中率）。
- Cross-tenant Leakage（跨租户泄露）必须为零。
- User Correction Success（用户纠正后是否正确更新）。
- Downstream Task Gain（对下游任务成功率的真实提升）。

离线构造多轮场景：建立偏好、改变偏好、撤回同意、跨会话询问、无关主题干扰、恶意记忆注入，验证整个生命周期。

## 11. 常见失败与排错

| 现象 | 常见根因 | 优先修复 |
|---|---|---|
| “记住了”但下次没用 | 未写入、召回过滤、上下文预算 | 查写入决策和召回 Trace |
| 总使用过期偏好 | 无版本/有效期/时间衰减 | 替代关系、TTL、冲突规则 |
| 记忆越来越多且质量下降 | 每轮全量写入、无整合遗忘 | 提高写入门槛、合并和淘汰 |
| 用户之间串数据 | 主体过滤晚于向量检索 | 检索前租户分区与权限过滤 |
| 模型引用不存在的“记忆” | 把模型推断当存储事实 | 要求 memory_id 与来源，缺失则不声称记得 |

## 12. 面试重点

### 12.1 Agent Memory 如何分类？

答题要点：按时长分工作、短期、长期；按功能分语义、情景、程序和偏好。分类的意义是使用不同写入、召回、冲突、权限和遗忘策略。

### 12.2 RAG 和 Memory 有什么区别？

答题要点：RAG 从组织或外部知识源为当前问题取证；Memory 从交互与任务沉淀跨时间有用的信息。二者可共享检索技术，但数据来源、生命周期、主体范围和治理不同。

### 12.3 怎样避免 Memory 污染？

答题要点：受控写入；区分用户确认、工具事实与模型推断；保存来源、置信度和版本；租户隔离；冲突与过期策略；程序性记忆先评测；用户可查看和删除。

### 12.4 为什么不能保存全部对话？

答题要点：成本和隐私风险上升，噪声降低召回精度，错误信息长期传播，且用户可能要求删除。原始日志、会话历史与长期记忆应有不同保留策略。

### 12.5 Checkpoint 是 Memory 吗？

答题要点：Checkpoint 是为了恢复当前运行的一致状态快照；Memory 是为未来任务复用的信息。前者强调执行一致性，后者强调长期效用、召回和遗忘。

## 13. 术语速查

| 英文 | 中文 | 含义 |
|---|---|---|
| Working Memory | 工作记忆 | 当前任务即时使用的信息 |
| Semantic Memory | 语义记忆 | 稳定事实与概念 |
| Episodic Memory | 情景记忆 | 带时间和上下文的事件 |
| Procedural Memory | 程序性记忆 | 方法、技能和操作经验 |
| Consolidation | 记忆整合 | 将多条事件抽象合并为稳定记忆 |
| Forgetting | 遗忘 | 过期、删除或压缩低价值记忆 |
| Provenance | 来源追踪 | 记忆从哪里产生、如何变化 |
| Staleness | 过时性 | 记忆不再反映当前事实 |

## 参考资料

- [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560)
- [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)
- [OpenAI Agents SDK：Sessions](https://openai.github.io/openai-agents-python/sessions/)
- [Anthropic：Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
