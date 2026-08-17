# AI Agent 工程师大厂面试手册（2026 版）

---

> 更新日期：2026-08-17  
> 适用目标：腾讯、字节跳动、阿里、百度、美团等公司的 AI Agent、LLM 应用及 AI 平台岗位。  
> 内容包括岗位能力判断、自测题、公开面经题、项目追问链和四周冲刺计划。  

> 面经来自公开岗位信息与候选人经验，不能视为公司官方题库。文中已区分公开面经题与根据岗位要求整理的预测练习题。

---

## 0. 使用方法

1. 先完成岗位能力定位，确定主投方向；
2. 对照 60 道自测题检查基础知识；
3. 优先准备腾讯专项和自己目标公司的公开面经题；
4. 每一道项目题至少追问到第三层：为什么、怎么测、失败怎么办；
5. 最后按照四周计划进行算法、系统设计和模拟面试。

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

## 2. 大厂面试应对方法

### 2.1 通常考察的五层

#### 第一层：基础编码

- Python/Go/Java 基础；
- 数据结构、并发、网络和数据库；
- 可测试、可维护代码。

#### 第二层：LLM 应用

- Prompt、RAG、Tool Calling；
- 幻觉、上下文、模型选择；
- Eval 和成本。

#### 第三层：Agent 系统

- Loop、State、Checkpoint；
- Memory、MCP、审批；
- 失败处理和安全。

#### 第四层：系统设计

- 高并发、长任务、队列；
- 多租户、隔离、可观测性；
- SLA、容量和成本。

#### 第五层：项目深挖

- 为什么这样选型；
- 失败过什么；
- 用什么数据证明改进；
- 如果流量扩大 100 倍怎么办；
- 如果重做会改什么。

### 2.2 项目回答模板

```text
背景：业务问题和用户是谁
约束：数据、延迟、成本、安全、规模
方案：架构和关键决策
难点：最棘手的两个问题
验证：评测集、线上指标、故障演练
结果：量化改进
复盘：失败、权衡、下一步
```

不要只说“使用 LangGraph + Milvus + FastAPI”。框架名不是能力证据。

### 2.3 系统设计回答顺序

1. 澄清用户、场景、规模和 SLO；
2. 定义核心 API 和数据模型；
3. 画主链路；
4. 拆控制面和运行面；
5. 解释状态、队列、幂等和恢复；
6. 加权限、安全和审计；
7. 加 Trace、Eval、指标和成本；
8. 找瓶颈、单点和降级方案；
9. 给出演进路线。

### 2.4 面试中的危险回答

- “用了向量数据库，所以不会幻觉”；
- “Temperature 设为 0，所以输出确定”；
- “用更大的模型就能解决”；
- “失败就重试三次”；
- “Prompt 里告诉模型不要越权”；
- “用了 Docker，所以沙箱安全”；
- “Multi-Agent 一定更智能”；
- “LLM Judge 打分高，所以可以上线”。

---

---

## 3. 60 道自测题

### LLM 与 Prompt

1. Self-Attention 在做什么？
2. Context Window 和最大输出 Token 有什么区别？
3. Temperature、Top-p 怎样影响采样？
4. 为什么长上下文不一定更准确？
5. 推理模型和普通生成模型怎样选择？
6. Prompt Caching 适用于什么输入结构？
7. Structured Output 为什么比 JSON 文本解析可靠？
8. 如何做模型升级回归？
9. 如何设计模型路由？
10. 怎样计算一次请求的完整成本？

### RAG

11. Sparse、Dense、Hybrid Search 的差异？
12. Embedding 的余弦相似度代表什么？
13. Chunk Size 如何选择？
14. Reranker 为什么有效？
15. Query Rewrite 什么时候会伤害召回？
16. 怎样处理 PDF 表格？
17. 怎样做文档 ACL？
18. Recall@K 与 MRR 的区别？
19. 怎样定位 RAG 错误阶段？
20. GraphRAG 的收益和成本？

### Agent

21. Agent 与 Workflow 的区别？
22. ReAct 的优缺点？
23. Plan-and-Execute 怎样重规划？
24. 怎样避免无限循环？
25. State 中应该保存什么？
26. Checkpoint 边界怎样确定？
27. 如何取消长任务？
28. Tool 结果太大怎么办？
29. Multi-Agent 何时适用？
30. 如何评测 Agent 任务成功率？

### Tool、MCP 与 Memory

31. Tool Calling 的完整协议循环？
32. 怎样保证副作用 Tool 幂等？
33. MCP Host、Client、Server 的关系？
34. Tool、Resource、Prompt 的区别？
35. stdio 与 Streamable HTTP 的区别？
36. 动态工具发现有什么风险？
37. 短期和长期 Memory 的区别？
38. Memory 如何更新和删除？
39. 怎样防止 Memory Poisoning？
40. 人工审批怎样恢复原运行？

### 平台与分布式系统

41. Control Plane 与 Data Plane 为什么分离？
42. At-least-once 下怎样实现业务幂等？
43. Lease 和 Heartbeat 的作用？
44. Outbox Pattern 解决什么问题？
45. 如何支持 SSE 断线续传？
46. 如何做多租户公平调度？
47. PostgreSQL、Redis、Qdrant 各自职责？
48. 怎样对长任务做 Backpressure？
49. 如何灰度发布 AgentVersion？
50. 如何设计模型供应商熔断和降级？

### 安全与可观测

51. Direct 和 Indirect Prompt Injection 的区别？
52. 为什么 Tool 授权必须在服务端？
53. SSRF 怎样防御？
54. 容器和 MicroVM 的隔离差异？
55. 哪些 Trace 字段可能泄露隐私？
56. 如何关联一次 Run 的所有 Span？
57. SLI、SLO、SLA 的区别？
58. 如何衡量每个成功任务的成本？
59. LLM Judge 有哪些偏差？
60. 如何构建 Agent Red Team 测试集？

能不看资料清楚回答其中 45 题，并用自己的项目证明至少 20 题，已经具备较强的应用/Agent 岗面试基础；平台岗还要继续加强分布式系统、隔离、SRE 和云基础设施。

---

---

## 4. 对现有能力体系的审查结论

### 4.1 总体评价

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

### 4.2 必须补充的能力

| 补充方向 | 为什么重要 | 掌握深度 |
| --- | --- | --- |
| Agent Skill 与动态能力路由 | 2026 年 Agent 越来越强调按需加载 Prompt、工具和知识，避免单一超长 Prompt | 必须能设计并实现 |
| Coding Agent / AI Coding | 大厂岗位开始直接要求熟悉 CLI、Skills、Agent 和 AI Coding 工具链 | 至少完成一个可演示项目 |
| 多模态与 GUI/Browser Agent | 手机、桌面、浏览器和企业流程 Agent 经常需要图像理解与界面操作 | 掌握架构、评测和安全 |
| 推理服务基础 | 腾讯部分岗位直接要求 vLLM、TensorRT-LLM、分布式推理或模型服务经验 | 应用岗理解，平台岗实操 |
| 微调与对齐边界 | 面试常问什么时候用 Prompt、RAG、SFT、LoRA、DPO，而不一定要求亲自训练大模型 | 能做选型和实验设计 |
| 计算机基础与算法 | 腾讯面经中经常把 Agent 与网络、并发、数据库、Linux、算法题混合考察 | 必须补强 |
| 数据闭环与产品指标 | 真正上线的 Agent 必须说明用户价值、失败样本、反馈闭环和量化收益 | 社招尤其重要 |

### 4.3 不建议继续扩张的内容

- 不要同时学习十个 Agent 框架，主线保持“原生模型 SDK + LangGraph/PydanticAI + MCP”即可；
- 不要为了面试堆砌 Multi-Agent，先证明单 Agent/Workflow 基线为什么不够；
- 不要把 CUDA、分布式训练、RLHF 全部列为 Agent 应用岗必修；
- 不要只背概念，每个核心模块必须有代码、测试、Trace 和量化结果；
- 不要把框架的示例项目直接写进简历，面试官会通过追问快速识别。

---

---

## 5. 腾讯等大厂公开面经题库

### 5.1 使用说明与可信度

- **A 类：公开候选人面经题**——来自公开面经整理，题目可能经过转述；
- **B 类：岗位要求推导题**——根据官方岗位描述和同类岗位高频考点设计；
- 公司、部门和面试官差异很大，不存在一份覆盖所有团队的“官方题库”；
- 复习时先自己回答，再用项目数据补充，不建议背固定话术。

### 5.2 腾讯专项：公开面经高频题（A 类）

#### Agent 与项目

1. 介绍你的 Agent 项目架构，你负责哪部分？
2. 多 Agent 是怎样编排的？为什么不用单 Agent？
3. Agent 在 thinking/decision 阶段怎样决定调用工具还是直接回答？
4. Agent 失败、中断或超时时怎样处理？重试是否安全？
5. Tool 已经执行成功，但 Agent 进程崩溃，恢复后怎么避免重复执行？
6. 你的项目有没有真实用户？为什么没有上线？
7. 项目效果怎么样？你亲自测过吗？使用了哪些指标？
8. Prompt 怎样设计？请现场写一个对应 Agent 的提示词。
9. Agent 和普通大模型问答的核心区别是什么？
10. Context Engineering 与 Memory 的关系是什么？
11. 上下文窗口受什么限制？超限后如何处理？
12. 在跨境汇款场景中，Agent 超时或失败时怎样保证资金安全？

#### RAG

13. RAG 知识库更新时怎样做到不停服？
14. 文档新增、修改和删除时，怎样避免召回旧版本？
15. 如何判断问题出在解析、召回、Rerank 还是生成？
16. 为什么选择当前 Embedding 模型和向量库？
17. Chunk 策略怎样确定？有没有做对比实验？
18. 多租户知识库怎样防止跨部门数据泄露？

#### 后端与系统

19. `select`、`poll`、`epoll` 的区别和适用场景？
20. TCP 三次握手、HTTPS 建连和 TIME_WAIT/CLOSE_WAIT？
21. 进程、线程和协程的区别？
22. Redis 分布式锁的失效和续租问题怎样处理？
23. 为什么使用 Redis Lua，而不是多个原子命令？
24. MySQL 隔离级别、MVCC、索引和深分页优化？
25. 消息队列怎样选型？如何保证不重复消费？
26. 如何管理多台服务器并实现负载均衡？
27. 如何设计 Agent 长任务的队列、Checkpoint 和恢复？

#### 算法与编码

28. DFS：岛屿最大面积；
29. 用两个栈实现队列，并讨论删除中间元素；
30. SQL：按商家统计已完成订单数量和金额并排序；
31. 常见链表、树、DFS/BFS、Top K、LRU 和并发题；
32. 算法岗可能要求手写多头注意力或反向传播。

### 5.3 腾讯岗位推导题（B 类）

腾讯当前公开岗位中出现了 C++/Linux/网络与多线程、Python asyncio、Agent 工程化、RAG、分布式推理、vLLM/TensorRT-LLM、图检索、大数据平台以及 CLI/Skills/AI Coding 等要求。针对这些要求，应补充练习：

1. 设计一个支持百万用户、长任务和人工审批的 Agent Runtime；
2. 怎样用 asyncio 控制模型和工具并发，并支持请求取消？
3. vLLM 为什么能提高吞吐？KV Cache 会怎样限制并发？
4. 如何为 Tool Calling 构建离线和在线评测集？
5. 如何实现 Skill Registry、版本管理和按需加载？
6. Coding Agent 如何保护用户未提交的修改？
7. 如何设计浏览器 Agent 的权限、登录和敏感操作审批？
8. 图检索、向量检索和关键词检索如何组合？
9. 如何把模型、Prompt、Tool、Skill 和 Eval 数据一起版本化？
10. 如果模型升级后工具调用正确率下降，怎样灰度和回滚？

### 5.4 字节跳动高频题（公开面经归纳，A 类）

1. 复杂 Agent 最主要的挑战是什么？
2. Agent 项目整体架构和最难的问题是什么？
3. Tool Calling 的 Bad Case 如何收集、分类和修复？
4. Memory 怎样区分短期、长期和用户画像？
5. Query Rewrite、Hybrid Retrieval、RRF、Rerank 各自解决什么问题？
6. Agentic RAG 与固定 RAG 有什么区别？
7. 法律/企业 RAG 如何保证引用和权限？
8. 上下文压缩如何避免丢失关键事实？
9. MCP 与 Skill 的区别？什么时候使用？
10. 怎样证明 RAG 准确率从 60% 提升到 85%，评测集是否泄漏？
11. Redis 常见数据结构、缓存一致性和热点问题；
12. Transformer、Attention、位置编码和 KV Cache；
13. 手撕链表、数组、树或字符串算法。

### 5.5 阿里/阿里云高频题（公开面经归纳，A 类）

1. ReAct 与 Plan-Execute-Replan 的区别和适用场景？
2. 为什么采用 Multi-Agent？整体 Workflow 是什么？
3. Agent 性能瓶颈怎样判断来自模型、检索还是工具？
4. 如何搭建 Agent/RAG 评价体系？
5. 知识库怎样构建，做了哪些性能和质量优化？
6. 上下文机制怎样设计？
7. 调试中遇到哪些问题，怎样把调试经验沉淀成数据或 Skill？
8. 平时怎样使用 AI Coding？AI 生成代码如何评审？
9. 连接建立涉及哪些超时？分别解决什么问题？
10. 红黑树、操作系统、网络和链表等后端基础题；
11. AI Coding 场景题：实现网关、日志分析或故障诊断系统。

### 5.6 百度、美团及其他大厂高频题（A/B 类混合）

1. Agent Skill 怎样开发？好的 Skill 包含什么？
2. 什么是 Plan Mode？工具调用完整流程是什么？
3. 向量相似度有哪些计算方法？
4. LoRA 的秩、Alpha、学习率怎样影响训练？
5. DPO 与 SFT/RLHF 的关系是什么？
6. 怎样设计多模态模型或视频生成模型的评测集？
7. 生产 RAG 如何实现百万文档、低延迟和实时更新？
8. 你如何使用 AI 辅助编码？举一个完整问题解决过程。
9. TCP/UDP、JVM/Go/Python、并发、数据库和中间件基础；
10. 设计一个任务管理、订单客服、日志分析或智能运营 Agent。

---

---

## 6. 大厂项目追问链：必须准备到第三层

### 6.1 Agent 项目追问链

```text
为什么需要 Agent？
  -> 为什么不能用固定 Workflow？
  -> 为什么选择单 Agent/Multi-Agent？
  -> Agent 如何选择工具？
  -> 工具失败怎么恢复？
  -> 如何避免重复副作用？
  -> 如何评测整条任务？
  -> 哪类失败最多？
  -> 优化后指标提升多少？
```

### 6.2 RAG 项目追问链

```text
为什么使用 RAG？
  -> 数据是什么格式、多少规模？
  -> 怎么解析和切分？
  -> 为什么选择该 Embedding？
  -> Hybrid Search 怎样融合？
  -> Rerank 为什么有效？
  -> ACL 在召回前还是后？
  -> Recall@K 怎么测？
  -> 最终答案错了怎样定位？
```

### 6.3 评测项目追问链

```text
评测集从哪里来？
  -> 是否覆盖真实流量？
  -> 标签由谁标？一致性如何？
  -> LLM Judge 如何校准？
  -> 是否有数据泄漏？
  -> 离线指标与线上指标是否相关？
  -> 新版本如何做显著性判断、灰度和回滚？
```

### 6.4 一份合格的项目证据

每个简历项目至少准备：

- 一张架构图；
- 一条完整请求 Trace；
- 一个失败案例分类表；
- 一组基线与优化后指标；
- 一次故障恢复或安全演练；
- 一个没有采用热门技术的明确理由；
- 一个“如果重做会改变什么”的复盘答案。

---

---

## 7. 分级题库与四周备考计划

### 7.1 第一优先级：必须熟练回答

- Agent 与 Workflow、RAG、普通 Chatbot 的区别；
- ReAct、Plan-and-Execute、Router；
- Tool Calling 循环、Schema、权限、幂等和审批；
- State、Checkpoint、重试、取消和恢复；
- Chunk、Embedding、Hybrid Search、Rerank；
- RAG 与 Agent 的评测指标和故障定位；
- Prompt Injection、数据越权和工具安全；
- Python/主语言、网络、数据库、并发；
- 自己项目的架构、指标、失败和取舍。

### 7.2 第二优先级：冲大厂和中高级

- Skill/Tool Registry 与动态路由；
- MCP、Memory 和长期任务；
- Model Gateway、限流、降级和成本；
- Durable Execution、Outbox、Lease、Heartbeat；
- 多租户、RBAC、Audit、Sandbox；
- vLLM、KV Cache、Continuous Batching；
- Coding/Browser/GUI Agent；
- LLM Judge 校准、红队和线上反馈闭环。

### 7.3 四周安排

#### 第 1 周：基础与腾讯专项

- 每天 2 道算法题；
- 复习网络、Linux、并发、数据库和 Redis；
- 完成腾讯 A 类题的口头回答；
- 把自己的态势 Agent 项目整理成 3 分钟与 10 分钟版本。

#### 第 2 周：RAG、Agent 和 Skill

- 实现一次混合检索与 Rerank 对比；
- 为 Agent 增加 request type、Skill 路由和动态工具集；
- 完成 Tool 幂等、超时、取消和 Checkpoint 测试；
- 整理 10 个真实 Bad Case。

#### 第 3 周：Eval、系统设计和安全

- 建立 50～100 条代表性评测集；
- 输出质量、延迟、Token、成本和工具成功率；
- 练习 Agent Runtime、企业知识库和模型网关三个系统设计题；
- 完成 Prompt Injection 与越权工具测试。

#### 第 4 周：项目包装与模拟面试

- 根据腾讯、字节、阿里题库各模拟两轮；
- 所有答案都追问“为什么、怎么测、失败怎么办”；
- 准备架构图、Trace、指标对比和复盘；
- 查漏补缺，不再新增框架。

---

---

## 8. 本次面经整理来源

### 官方岗位参考

- [腾讯招聘：大模型（Agent）应用相关岗位](https://careers.tencent.com/jobdesc.html?postId=2084242179768893440)
- [腾讯招聘：LLM 推理优化、图构建与检索相关岗位](https://careers.tencent.com/jobdesc.html?postId=1934984932162117632)
- [腾讯招聘：混元强化训练框架研发工程师](https://careers.tencent.com/jobdesc.html?postId=2061810859658887168)
- [腾讯招聘：AI Coding、Skills 与推理框架相关岗位](https://careers.tencent.com/jobdesc.html?postId=2083008229050335232)

### 公开面经与题库参考

- [牛客：腾讯/百度大模型与 Agent 面经汇总（2026-04）](https://www.nowcoder.com/discuss/878600528970735616)
- [牛客：腾讯公司面试经验入口](https://www.nowcoder.com/enterprise/138/interview)
- [牛客：阿里、蚂蚁、字节 Agent 开发面经汇总](https://www.nowcoder.com/discuss/877151327091027968)
- [牛客：阿里云 AI Agent 平台研发一面](https://www.nowcoder.com/feed/main/detail/476de2514a3c457c8be774623549f4b1)
- [腾讯云开发者社区：字节 AI Agent 二面题目解析](https://cloud.tencent.com/developer/article/2673729)
- [Datawhale Hello-Agents：LLM、VLM 与 Agent 面试题](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra01-%E9%9D%A2%E8%AF%95%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93.md)
- [Agent Interview Hub：公开面经索引](https://github.com/Zchary1106/agent-interview-hub/blob/main/%E9%80%9A%E7%94%A8%E7%9F%A5%E8%AF%86/%E6%9C%80%E6%96%B0AI-Agent%E9%9D%A2%E7%BB%8F%E7%B4%A2%E5%BC%95.md)

> 建议每两周检查一次公开面经，每季度检查一次官方岗位要求。新增题目时记录公司、岗位、轮次、日期、来源和考察模块；来源不明的题目统一标为“预测练习题”，避免把二次整理内容误认为真实面经。
