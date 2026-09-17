# LLM 基础与推理高频面试题

> 返回：[AI Agent 面试题库](README.md)｜配套知识：[LLM 基础](../知识体系/LLM基础.md)

## 1. Transformer 的核心结构是什么？为什么比 RNN 更适合大模型训练？ `★★★★★`

### 一、Transformer 的核心结构

Transformer 会先将输入文本转换为 Token，再通过 Embedding（词向量）和位置编码转换成带有位置信息的向量，然后经过多个重复的 Transformer Block，最后由输出层预测下一个 Token。

整体流程可以概括为：

输入文本 → Token 化 → Embedding + 位置信息 → 多层 Transformer Block → 输出层 → 下一个 Token

一个 Transformer Block 主要包含以下组件：

1. **多头自注意力机制（Multi-Head Self-Attention）**

    自注意力机制通过 Q、K、V 计算不同 Token 之间的关联程度，使每个 Token 能够结合上下文中的其他信息更新自身表示。

    多头注意力会从不同角度学习关系，例如实体关系、语义关系、位置关系和因果关系。

2. **前馈神经网络（Feed Forward Network，FFN）**

    Attention 负责汇集不同 Token 之间的信息，FFN 则对每个 Token 已经汇集到的隐藏特征进行进一步非线性加工。

    可以简单理解为：
    - Attention 负责建立信息之间的关系；
    - FFN 负责加工已经建立关系的信息。

3. **残差连接（Residual Connection）**

    将模块处理结果与原始输入相加，为原始信息和梯度提供直接传递通道，减少深层网络中的信息丢失和梯度消失问题。

4. **层归一化（Layer Normalization，LayerNorm）**

    将每个 Token 的隐藏向量调整到相对稳定的数值范围，提高深层模型训练的稳定性。

以常见的 Pre-Norm 结构为例：

x₁ = x + Attention(LayerNorm(x))

x₂ = x₁ + FFN(LayerNorm(x₁))

多个 Transformer Block 处理完成后，最终隐藏向量会经过输出层映射成词表中所有 Token 的概率，从中选择下一个 Token，并不断重复该过程，最终生成完整答案。

---

### 二、Transformer 为什么比 RNN 更适合大模型训练

#### 1. Transformer 可以进行并行训练

RNN 必须按照时间顺序依次处理 Token：

Token₁ → Token₂ → Token₃ → Token₄

后一个 Token 必须等待前一个 Token 计算完成，因此难以充分利用 GPU 进行大规模并行训练。

Transformer 在训练阶段可以同时处理序列中的多个 Token，通过注意力矩阵计算它们之间的关系，因此训练效率和硬件利用率更高。

#### 2. Transformer 更容易处理长距离依赖

RNN 中，前面的信息需要经过多个时间步骤才能传递到后面。序列较长时，容易出现早期信息丢失和梯度消失问题。

Transformer 的自注意力机制可以让任意两个 Token 直接建立联系。例如句首信息和句尾信息不需要经过中间所有 Token 逐步传递。

#### 3. Transformer 更容易扩展模型规模

Transformer 主要由矩阵计算组成，适合使用 GPU、TPU 和分布式集群训练。模型可以通过增加以下内容扩展能力：

- Transformer 层数；
- 隐藏向量维度；
- 注意力头数量；
- FFN 参数规模；
- 训练数据量。

这使Transformer更容易扩展到数十亿甚至更大参数规模。

#### 4. 深层训练更加稳定

Transformer 使用 LayerNorm 和残差连接，让信息与梯度能够稳定地通过多层网络，降低深层模型训练困难的问题。

#### 5. 多头注意力能够同时学习多种关系

不同注意力头可以关注不同类型的信息，使模型同时学习语法、实体、位置、上下文和逻辑关系，比RNN单向逐步传递隐藏状态更灵活。

---

### 三、Transformer 的局限性

Transformer 并不是所有方面都优于RNN：

- 标准注意力机制的计算量和显存消耗通常随序列长度呈平方增长；
- 训练时可以并行处理Token，但生成答案时仍然需要逐Token生成；
- 上下文过长时，需要使用上下文压缩、分段处理或更高效的注意力机制。

---

### 四、实际案例：态势智能体

态势智能体的一次输入可能同时包含任务数据、车辆状态、地图工具结果和RAG证据。

例如：

- 一号车辆剩余能源20%；
- 当前路线长度18公里；
- RAG资料说明能源低于30%时不建议执行长距离机动；
- 用户要求分析任务风险。

Transformer 的自注意力机制可以直接建立“一号车辆—能源20%—低于30%—18公里路线—机动风险”之间的关系，FFN进一步加工这些信息，最终由输出层逐Token生成风险结论。

如果使用RNN，这些信息需要按照输入顺序逐步传递，输入内容较长时，前面的车辆状态可能在传递过程中逐渐弱化。Transformer可以通过自注意力直接关注相关Token，因此更适合处理Agent中较长的业务数据、RAG证据和工具结果。

---

### 五、面试口述版

Transformer 的核心是由多层 Transformer Block 组成，每个Block主要包含多头自注意力、前馈神经网络、残差连接和LayerNorm。Attention负责建立不同Token之间的关系，FFN负责进一步加工每个Token的隐藏特征，残差连接和LayerNorm则保证深层网络训练稳定，最后由输出层逐Token生成答案。

相比RNN，Transformer最大的优势是训练阶段可以并行处理Token，并且能够通过自注意力直接建立长距离依赖，更容易利用GPU和分布式集群扩展模型参数与训练数据规模。RNN需要按照时间顺序逐步计算，难以并行，而且长序列中容易丢失早期信息。因此，Transformer比RNN更适合训练大语言模型。

## 2. Scaled Dot-Product Attention（缩放点积注意力）为什么除以 `sqrt(d_k)`？多头有什么作用？ `★★★★☆`

### 标准回答

注意力计算可写为：

```text
Attention(Q, K, V) = softmax(QKᵀ / sqrt(d_k))V
```

当查询和键各维近似独立且方差相近时，点积方差会随维度 `d_k` 增大。数值过大会让 Softmax（归一化指数函数）进入饱和区，梯度变小；除以 `sqrt(d_k)` 用于稳定尺度。

Multi-Head Attention 把表示投影到多个子空间，让不同头学习不同位置关系或特征，再拼接投影。它提升表达能力，但不能把每个头机械解释成固定语义；头的功能是训练后涌现且可能冗余。

## 3. Token、Embedding 和 Positional Encoding 分别解决什么问题？ `★★★★☆`

### 标准回答

Tokenizer（分词器）把原始文本映射为离散 Token ID；Embedding（嵌入）把 ID 映射成连续向量；位置编码向模型注入顺序信息，因为纯注意力本身对输入排列没有天然顺序感。

不同模型的 Tokenizer 不同，同一句话的 Token 数可能不同，成本和上下文长度也不能直接按字符估算。位置方法包括原始正弦位置编码、Learned Position Embedding（可学习位置嵌入）和 RoPE（旋转位置编码）等。长上下文扩展不能只修改一个长度参数，还涉及位置外推、训练分布和注意力利用能力。

## 4. Pretraining、SFT、RLHF 和 DPO 有什么区别？ `★★★★★`

### 标准回答

- Pretraining（预训练）：在大规模语料上学习下一个 Token 预测，获得通用语言和知识能力。
- SFT（监督微调）：用高质量“输入—期望输出”样本训练指令遵循和任务格式。
- RLHF（基于人类反馈的强化学习）：收集人类偏好，训练 Reward Model（奖励模型），再通过 PPO 等强化学习方法优化策略。
- DPO（直接偏好优化）：直接使用偏好对优化策略与参考策略的相对概率，不显式训练奖励模型并运行在线 RL，流程更简单。

它们解决的问题不同，也都不能保证事实正确、安全或业务规则绝对遵守。实际训练还要防止 Reward Hacking（奖励投机）、灾难性遗忘、数据污染和能力回退。

## 5. Temperature 和 Top-p 如何影响生成？低温度能消除幻觉吗？ `★★★★★`

### 标准回答

Temperature（温度）对 Logit（未归一化分数）缩放：温度越低，分布越尖锐，结果通常更稳定；越高则更随机。Top-p（核采样）只在累计概率达到 `p` 的最小候选集合中采样。

事实抽取、分类、工具参数通常使用偏保守配置，创意任务可提高随机性。但低温度不能消除幻觉：如果模型缺少事实、上下文错误或问题本身歧义，它仍可能稳定地输出错误答案。事实性需要通过 RAG、工具查询、引用、校验和拒答治理。

## 6. 为什么大模型会产生 Hallucination（幻觉）？如何系统治理？ `★★★★★`

### 标准回答

语言模型优化的是给定上下文下的 Token 概率，不是从事实数据库读取真值。训练知识过期、问题超出知识范围、上下文缺失或冲突、采样随机性以及模型过度迎合都可能产生幻觉。

治理要分层：

1. 用 RAG 或工具提供可信且最新的事实。
2. 对证据做权限、版本、相关性和充分性检查。
3. 要求关键声明引用来源并允许拒答。
4. 对声明做规则、NLI（自然语言推断）或模型核验。
5. 高风险结果由确定性系统或人工确认。
6. 用真实失败集持续评测，而不是只在 Prompt 中写“不要幻觉”。

## 7. Prompt、RAG 和 Fine-tuning（微调）怎样选择？ `★★★★★`

### 标准回答

- Prompt 适合改变任务说明、输出格式和少量示例，更新快、成本最低。
- RAG 适合频繁变化、私有、规模大且需要引用的知识。
- Fine-tuning 适合稳定行为、风格、领域表达、分类边界或让模型学习大量示例中的模式。

不应主要依赖微调灌入需要频繁更新的事实，也不能期待 RAG 自动改变模型稳定行为。常见组合是：Prompt 定义任务，RAG 提供证据，微调改善行为或领域适配，工具查询实时状态。

## 8. Embedding（嵌入）是什么？余弦相似度、点积和欧氏距离怎样选？ `★★★★★`

### 标准回答

Embedding 把文本、图像等对象映射到稠密向量，使语义相近的对象在向量空间中更接近。Cosine Similarity（余弦相似度）比较方向；Dot Product（点积）同时受方向和模长影响；Euclidean Distance（欧氏距离）比较绝对距离。

如果向量已 L2 归一化，余弦相似度与点积排序等价，欧氏距离排序也存在单调关系。选择必须与模型训练目标和向量数据库实现一致。Embedding 适合候选召回，不等于精确事实判断；生产检索通常结合 BM25、元数据过滤与 Reranker。

## 9. Structured Output（结构化输出）与普通 JSON Prompt 有什么差别？ `★★★★☆`

### 标准回答

普通 JSON Prompt 只是自然语言要求，模型可能输出多余文本、缺字段或类型错误。原生 Structured Output 或受约束解码会按照 JSON Schema 限制生成空间，提高语法和结构符合率。

但结构合法不等于业务正确。执行前仍要做字段范围、实体存在性、权限、额度和风险校验。例如金额是整数，不代表该用户有权退款这个金额。

## 10. KV Cache（键值缓存）是什么？为什么能加速自回归生成？ `★★★★★`

### 标准回答

自回归模型每生成一个新 Token，都需要让新查询关注之前所有位置。历史 Token 在各层计算得到的 Key 和 Value 不会改变，因此可以缓存，后续只计算新 Token 的投影并与缓存交互，避免重复计算整个前缀。

KV Cache 会随层数、序列长度、并发和 KV Head 数增长，占用大量显存。常见优化包括 PagedAttention（分页注意力）、Continuous Batching（连续批处理）、Prefix Caching（前缀缓存）、量化、Grouped-Query Attention（分组查询注意力）和上下文压缩。

## 11. Prefill 与 Decode 阶段有什么区别？ `★★★★☆`

### 标准回答

Prefill（预填充）一次处理输入 Prompt，通常计算量大且能较好并行；Decode（逐词生成）每一步只生成少量 Token，依赖之前结果，通常更受内存带宽、KV Cache 访问和调度影响。

因此要分别观察 Time to First Token（首 Token 延迟）和 Time per Output Token（每输出 Token 时间）。长输入优化主要影响 Prefill；批处理、KV 管理和推测解码等更直接影响 Decode。只看端到端平均延迟无法定位瓶颈。

## 12. 如何优化大模型在线推理的吞吐、延迟和成本？ `★★★★★`

### 标准回答

先建立质量和性能基线，再按瓶颈选择措施：

- 模型层：量化、蒸馏、更小模型路由、稀疏或 MoE（混合专家）模型。
- Kernel（算子）层：FlashAttention、融合算子和高效矩阵计算。
- Serving（服务）层：Continuous Batching、PagedAttention、Prefix Cache、并行策略和请求调度。
- 应用层：缩短上下文、限制输出、缓存、批量工具、减少无效重试。

吞吐和单请求延迟经常冲突：更大批次提高吞吐，但增加排队延迟。量化降低显存和计算成本，也可能损失质量。最终应比较每个成功任务的总成本、P95/P99 延迟和业务成功率，而不是只看 Tokens/s。

## 13. LoRA 为什么能用较少参数完成微调？ `★★★★★`

### 标准回答

LoRA（低秩适配）冻结原模型权重 `W`，只学习一个低秩增量 `ΔW = BA`，其中秩 `r` 远小于原矩阵维度。训练参数、优化器状态和显存明显减少，不同任务还可保存为独立 Adapter（适配器）。

它基于“任务适配所需权重变化具有较低内在秩”的假设。秩、目标层、学习率和数据质量仍需评测；LoRA 降低训练成本，不保证推理零开销，也不适合用来频繁更新可检索事实。

## 14. 大模型量化是什么？INT8、INT4 的收益和风险有哪些？ `★★★★☆`

### 标准回答

量化用更低位宽表示权重、激活或 KV Cache，从而降低显存、带宽和部分计算成本。PTQ（训练后量化）无需完整再训练、部署方便；QAT（量化感知训练）在训练时模拟量化，通常能更好恢复精度但成本更高。

位宽越低不代表端到端越快，收益取决于硬件与 Kernel 支持；异常值、不同层敏感度和校准数据都会影响质量。必须分别评估任务成功率、困惑度或业务指标、吞吐、尾延迟和显存。

## 15. MHA、MQA 和 GQA 有什么区别？ `★★★★☆`

### 标准回答

MHA（多头注意力）每个查询头拥有独立 Key/Value 头；MQA（多查询注意力）让多个查询头共享一组 Key/Value，大幅减少 KV Cache 和解码带宽，但可能损失表达能力；GQA（分组查询注意力）让一组查询头共享 Key/Value，是质量与效率之间的折中。

它们主要影响推理阶段 KV Cache 规模和 Decode 性能。不能只依据名称判断实际速度，还要结合模型层数、头数、序列长度、并发、Kernel 和硬件测试。

## 16. MoE（混合专家）模型为什么能扩大参数量但不同比例增加单 Token 计算？ `★★★★☆`

### 标准回答

MoE 层包含多个 Expert（专家）网络，Router（路由器）通常只为每个 Token 选择 Top-k 个专家，因此总参数量可很大，但单 Token 只激活其中一小部分。

难点包括负载不均、热点专家、跨设备 All-to-All 通信、路由稳定性、容量溢出和训练辅助损失。推理中总参数仍要存储或分布，显存与通信不会因为稀疏激活自动消失；实际收益取决于并行布局和批量。

## 参考资料

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [Training Language Models to Follow Instructions with Human Feedback](https://arxiv.org/abs/2203.02155)
- [Direct Preference Optimization](https://arxiv.org/abs/2305.18290)
- [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685)
- [GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers](https://arxiv.org/abs/2210.17323)
- [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://arxiv.org/abs/2305.13245)
- [Switch Transformers](https://arxiv.org/abs/2101.03961)
- [vLLM: Easy, Fast, and Cheap LLM Serving with PagedAttention](https://arxiv.org/abs/2309.06180)
- [FlashAttention](https://arxiv.org/abs/2205.14135)
