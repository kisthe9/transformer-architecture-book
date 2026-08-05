# 外部知识研究

> 本文件是 V2 全书的外部知识索引：竞品定位、论文溯源、前沿扫描、工程对标、教学借鉴、知识缺口。
> 每章动笔前的"单章外部知识研究"摘要追加在文末对应章节段落中。
> 检索时间基线：2026-07（首轮研究）。

---

## 竞品分析

| 书名 | 作者 | 读者定位 | 结构特点 | V2 的差异化价值 |
|------|------|----------|----------|-----------------|
| *Build a Large Language Model (From Scratch)*（Manning, 2024） | Sebastian Raschka | 会 Python/PyTorch 的 ML 学习者 | 从零手写 GPT-2 级模型：分词→注意力→训练→微调，代码驱动，动手性极强 | Raschka 聚焦**训练**且用玩具尺寸（124M GPT-2）；V2 聚焦**推理**且全程用真实尺寸（70B），覆盖它完全不讲的 KV Cache 管理、PagedAttention、Continuous Batching、带宽瓶颈分析 |
| *Hands-On Large Language Models*（O'Reilly, 2024） | Jay Alammar & M. Grootendorst | 应用开发者 | 图解风格 + 应用导向（RAG、Agent、微调），可视化质量业界最佳 | 它重"用模型"轻"模型内部"——注意力只讲概念层；V2 深入每个矩阵的形状与账本，回答它跳过的"为什么是这个设计" |
| *Natural Language Processing with Transformers*（O'Reilly, 2022） | Tunstall, von Werra, Wolf (HF 团队) | NLP 工程师 | 围绕 HF Transformers 库的任务流（分类/NER/摘要），库 API 驱动 | 强绑定库版本，架构原理是配角；V2 不绑定框架、以原理为主角，且覆盖 2022 后的推理工程革命（vLLM 时代） |
| *Transformers for Natural Language Processing*（Packt, 2022） | Denis Rothman | 广泛 AI 从业者 | 模型动物园式巡览（BERT/GPT/T5/ViT…），广度优先 | 广而浅、章节间独立；V2 反其道：单一模型（LLaMA-2 70B）单一主线钻到底，深度优先 |
| *Speech and Language Processing* 3rd ed.（在线草稿） | Jurafsky & Martin | 高校课程 | 学术教科书，覆盖全 NLP，习题严谨 | 学术视角、无推理工程；V2 是工程师视角，把"论文→生产系统"的最后一公里讲透 |

**V2 生态位结论**：市面书籍分两极——"从零训练玩具模型"（Raschka）与"调用模型做应用"（Alammar）。**中间地带空缺：以真实生产尺寸模型为标本、从算法一路讲到 GPU 带宽的"推理解剖书"**。V2 占据这个生态位，读者选择理由：读完 Raschka 仍不懂 vLLM 为什么快，读完 Alammar 仍不懂显存为什么爆——V2 回答这两个问题。

## 核心论文清单

| 论文 | 作者/年份 | 核心贡献 | 关联章节 |
|------|-----------|----------|----------|
| *Attention Is All You Need* | Vaswani et al., 2017 | Transformer 架构与缩放点积注意力 | ch0, ch2（正弦编码）, ch3 |
| *LLaMA 2: Open Foundation and Fine-Tuned Chat Models* | Touvron et al., 2023 | 本书基准模型；70B 参数表（8192/80L/64H/GQA-8/28672/V=32000，已经 HF config 交叉验证 ✓） | 全书 |
| *Neural Machine Translation of Rare Words with Subword Units* | Sennrich et al., 2016 | BPE 用于 NLP 分词 | ch1 |
| *SentencePiece* | Kudo & Richardson, 2018 | LLaMA 分词器实现基础 | ch1 |
| *RoFormer: Enhanced Transformer with Rotary Position Embedding* | Su et al., 2021 | RoPE：旋转注入相对位置 | ch2 |
| *YaRN: Efficient Context Window Extension of LLMs* | Peng et al., 2023 | 频率缩放式长度外推 | ch2（延伸） |
| *Lost in the Middle: How Language Models Use Long Contexts* | Liu et al., TACL 2024 | 长上下文召回的 U 形退化曲线 | ch2 §2.7（第二层天花板的实证） |
| *Fast Transformer Decoding: One Write-Head is All You Need* | Shazeer, 2019 | MQA | ch4 |
| *GQA: Training Generalized Multi-Query Transformer Models* | Ainslie et al., EMNLP 2023 | GQA 与消融实验（8 组质量近无损） | ch4 |
| *GLU Variants Improve Transformer* | Shazeer, 2020 | SwiGLU | ch5 |
| *Transformer Feed-Forward Layers Are Key-Value Memories* | Geva et al., EMNLP 2021 | FFN 键值存储视角 | ch5 |
| *Locating and Editing Factual Associations in GPT*（ROME） | Meng et al., NeurIPS 2022 | 知识定位于 FFN 的实证 | ch5（延伸） |
| *Deep Residual Learning*（ResNet） | He et al., CVPR 2016 | 残差连接起源 | ch6 |
| *Root Mean Square Layer Normalization* | Zhang & Sennrich, NeurIPS 2019 | RMSNorm | ch6 |
| *On Layer Normalization in the Transformer Architecture* | Xiong et al., ICML 2020 | Pre-Norm vs Post-Norm 理论分析 | ch6 |
| *The Curious Case of Neural Text Degeneration* | Holtzman et al., ICLR 2020 | top-p 采样；贪心退化分析 | ch7 |
| *Hierarchical Neural Story Generation* | Fan et al., ACL 2018 | top-k 采样 | ch7 |
| *Efficiently Scaling Transformer Inference* | Pope et al., MLSys 2023 | Prefill/Decode 特性、推理显存与并行分析 | ch8, ch9 |
| *Efficient Memory Management for LLM Serving with PagedAttention* | Kwon et al., SOSP 2023 | vLLM/PagedAttention；传统方式利用率 20~40% 实测 | ch9 |
| *Megatron-LM: Training Multi-Billion Parameter Models* | Shoeybi et al., 2019 | 张量并行的列切/行切方案 | ch9（新增节） |
| *Orca: A Distributed Serving System* | Yu et al., OSDI 2022 | Continuous Batching（iteration-level scheduling）首创 | ch10 |
| *FlashAttention: Fast and Memory-Efficient Exact Attention* | Dao et al., NeurIPS 2022 | IO 感知注意力内核 | ch10 |
| *Roofline: An Insightful Visual Performance Model* | Williams et al., CACM 2009 | 算术强度分析框架 | ch10 |
| *Fast Inference from Transformers via Speculative Decoding* | Leviathan et al., ICML 2023 | 投机解码 | ch10（新增节） |

## 最新进展（2024-2026）

| 方向 | 代表工作 | 纳入方式 | 理由 |
|------|----------|----------|------|
| MLA（低秩 KV 压缩） | DeepSeek-V2/V3 (2024) | ch4 & ch9 正文短节 + 延伸阅读 | 是 GQA 叙事线（"KV 还能再压吗"）的自然下一步，读者面试/选型高频遇到 |
| MoE 稀疏化 FFN | Mixtral (2024), DeepSeek-V3 671B/37B 激活 | ch5 正文短节（"FFN 的下一步"）+ 附录 | FFN=知识库叙事的自然延伸；但本书主线是稠密模型，不展开路由细节 |
| Multi-Token Prediction | DeepSeek-V3 (2024) | ch10 投机解码节内一笔带过 | 与投机解码同属"绕过逐字瓶颈"，合并叙述避免碎片化 |
| 状态空间模型 | Mamba/Mamba-2, RWKV | 附录延伸阅读 | 是 Transformer 的替代而非组件，展开会破坏主线；给读者一个"注意力 O(n²) 的出路"指针 |
| 长上下文扩展 | YaRN, NTK-aware, ALiBi | ch2 工程视角节 + 延伸阅读 | RoPE 频率手术是 ch2 相对性结论的直接应用，正文点到机制即可 |
| 上下文窗口上限归属 | Lost in the Middle (Liu 2024)；推理服务 max_model_len 配置实践 | ch2 §2.7 三层天花板 + ch9 §9.1 账本回收 | "是模型限制还是服务限制"是读者高频疑问；三层框架（量程/训练分布/开放成本）正面回答，200K→65 GB 账本作为 ch2→ch9 新伏笔 |
| 量化 | GPTQ, AWQ, GGUF/llama.cpp, FP8 KV | ch9/ch10 正文短节 | 显存与带宽两章的"第三味药"，讲原理（低精度=省容量+等效带宽翻倍）不讲工具链细节 |
| FlashAttention-3 | Dao et al., 2024 | ch10 延伸阅读 | Hopper 特化，原理与 FA1/2 一脉相承，正文讲通用原理即可 |
| 推理引擎演进 | vLLM V1 重构 (2025)：调度器简化、近零开销 prefix caching | ch10 正文事实基准 | 已验证：V1 重构确认"控制平面开销"是真实工程痛点，支撑 ch10 的 CPU/GPU 分工叙事 |

## 工程实现对标

| 项目 | 对标内容 | 关联章节 |
|------|----------|----------|
| HF Transformers `modeling_llama.py` | LlamaAttention/LlamaMLP/LlamaRMSNorm 的真实前向顺序、`repeat_kv` 的 GQA 实现、RoPE 的 rotate_half 写法 | ch2~ch6 代码锚点的正确性基准 |
| HF `config.json` (Llama-2-70b) | hidden_size=8192, num_hidden_layers=80, num_attention_heads=64, num_key_value_heads=8, intermediate_size=28672, vocab_size=32000 ✓ | 全书数值 |
| vLLM（V0 PagedAttention 内核 + V1 引擎） | 块表/块池/CoW 实现；调度循环；Continuous Batching | ch9, ch10 |
| PyTorch `scaled_dot_product_attention` | 因果掩码语义（is_causal）、缩放约定 | ch2 |
| llama.cpp | GGUF 量化分级、CPU 侧推理的带宽账本 | ch9/ch10 量化节佐证 |
| tiktoken / SentencePiece 演示页 | 分词练习的可操作工具 | ch1 练习 |

## 教学案例借鉴

| 资源 | 值得借鉴的手法 | V2 如何应用 |
|------|---------------|-------------|
| 3Blue1Brown《Attention in transformers》 | 把嵌入向量讲成"高维空间中的方向"，用"名词被形容词更新"演示注意力的语义位移 | ch1 分布式表示、ch2 "向量互相塑造"的叙述框架（重新表达） |
| Jay Alammar《The Illustrated Transformer》 | 一步一图、每步只引入一个新符号 | V2 的 ASCII 图规范：每图只讲一个转换，形状必标 |
| Karpathy《Let's build GPT》/nanoGPT | 代码即讲义：变量名与公式符号一一对应；先跑通再优化 | E1 代码锚点的设计规范（≤30 行纯 NumPy、符号对齐） |
| The Annotated Transformer (Harvard NLP) | 论文原文与代码并排对照 | 附录 A 把全书代码锚点串成完整玩具推理器的组织方式 |
| Transformer Explainer (Georgia Tech, 2024) | 交互式温度/采样滑杆直观展示分布变形 | ch7 用数值表格模拟"滑杆"效果（同一 logits 三档温度对照） |
| Brendan Bycroft LLM Visualization | 3D 展示每个张量的真实形状比例 | 强化 V2 "量纲驱动"原则：所有图中矩阵按真实长宽比示意 |

**超越点**：以上资源全部止步于单次前向或训练视角，无一覆盖"生成循环之后"的工程半场（KV Cache 管理、调度、带宽）。V2 用同等教学质量把后半场补齐——这是教学生态位的空白。

## 知识缺口分析

通过外部研究发现的、早期研究笔记中缺失或不够深入的主题：

- **张量并行**：内部素材只在显存估算中隐含 TP=4，无机制讲解。外部（Megatron-LM 论文）显示列切/行切 + AllReduce 是 70B 落地的前提。→ V2 第 9 章开头新增"把 70B 塞进多张卡"节。
- **MLA/DeepSeek 线**：内部素材停在 GQA。外部研究显示 2024 后 KV 压缩主战场已移到 MLA（低秩联合压缩），且 DeepSeek-V3 证明其规模化。→ V2 在 ch4/ch9 各设短节衔接。
- **采样参数全景**：内部素材只覆盖 temperature/top-p。真实 API 还有 top-k、repetition/presence/frequency penalty、min-p（2024 起流行）。→ V2 第 7 章补全。
- **投机解码**：内部素材未覆盖。外部研究显示它已是生产标配（含 DeepSeek MTP 变体）。→ V2 第 10 章新增节。
- **vLLM V1 演进**：内部素材基于 vLLM 早期架构。2025 年 V1 重构（调度器简化、近零开销 prefix caching）需反映在 ch10 的事实表述中，避免出版即过时。
- **BPE 训练过程演示**：内部素材只给思想不给过程。外部教学案例（Raschka ch2）显示"手推一次合并"教学效果最好。→ V2 第 1 章给最小合并演示。

---

## 单章研究记录（写作时逐章追加）

### ch0 · 鸟瞰（第 2 轮）
- **论文验证**：全链路站点划分与 Pope et al. (MLSys 2023) 的推理流水线划分一致；LLaMA-2 70B 参数与 HF config 复核一致（见上表 ✓）。
- **工程验证**：外壳流程（HTTP→排队→调度→流式）对标 vLLM V1 引擎的 API server → EngineCore 分层（2025 blog 已确认该分层现实存在）。
- **教学验证**：借鉴 Alammar "一步一图"与 Bycroft "真实比例"原则设计全景解剖图；避免 3B1B 式从嵌入空间切入（对本书读者太抽象），改用工程师熟悉的"一次 curl 请求"切入。
- **前沿补充**：ch0 只挂地图不展开前沿；在"外壳"一段为 ch9/ch10 的量化/投机解码留钩子。
- **缺口填补**：无（ch0 不涉及已识别缺口的主题）。
- **检查点 A 后追认**：章节指针已按新章序（ch2 位置、ch3 注意力）同步修正。

### ch1 · 分词与嵌入（第 3 轮）
- **论文验证**：BPE 合并算法步骤与 Sennrich et al. (ACL 2016) 一致（统计相邻对频率→合并最高频→迭代）；LLaMA-2 分词器为 SentencePiece BPE、词表 32000，与 Touvron et al. 2023 及 Kudo & Richardson 2018 一致；嵌入矩阵 (32000, 8192) ≈ 2.62 亿参数。
- **工程验证**：外部检索确认 SentencePiece 启用 byte fallback：词表外字符退化为字节 token（汉字最多 3 个）、永无 UNK——支撑正文"任何字符串都能表示"的表述与教学简化声明的准确性；Llama 3 已换 tiktoken 系分词器（正文以 LLaMA-2 为准，延伸阅读提及演进）。
- **教学验证**：借鉴 Raschka ch2 "手推一次合并"的教学手法（重新设计为英文最小语料演示）；借鉴 3B1B "向量=高维方向"的叙述框架讲分布式表示（重新表达）。
- **前沿补充**：超大词表趋势（Llama 3: 128K、Qwen2: 152K）作为"词表大小权衡"节的现实参照写入正文短段；中文分词效率差异作为工程视角。
- **缺口填补**：已识别缺口"BPE 合并过程演示"——本章用最小语料完整手推一轮合并。

### ch2 · 位置编码（第 4 轮）
- **论文验证**：RoPE 频率公式 $\theta_j = 10000^{-2j/d_k}$、仅作用于 Q/K、相对性性质 $\langle R_m q, R_n k\rangle = \langle q, R_{n-m} k\rangle$ 均与 Su et al. 2021 一致；第一代正弦加法方案出自 Vaswani 2017 §3.5；LLaMA-2 用 base=10000。
- **工程验证**：外部检索确认 HF `rotate_half` 用"前后半分组"而非原论文"奇偶交错配对"（GPT-J 式）——两者是等价的维度置换（权重随之置换，数学结果相同），正文代码用交错配对以贴合公式，并加注说明 HF 实现差异；RoPE 在每层注意力内部、投影之后、点积之前施加。
- **教学验证**：参考 EleutherAI 博客与主流教程的"先二维后高维"路径；里程表多转盘类比为原创表达并声明边界。
- **前沿补充**：长上下文三条路线入正文短节：位置插值/NTK/YaRN（频率手术）与"直接抬 base"（Llama 3 将 base 提至 500000）；ALiBi 作为旁支在延伸阅读。
- **缺口填补**：已识别缺口"长上下文扩展机制"——本章工程视角节展开。
- **追加（上下文窗口归属）**：新增 §2.7"三层天花板"，正面回答"上下文窗口是模型限制还是推理服务限制"：①RoPE 量程（§2.6 频率手术）②训练分布有效窗口（引 Liu et al. TACL 2024 的 lost in the middle U 形曲线）③服务开放成本（200K→约 65 GB KV 剧透，ch9 §9.1 用 ch4 公式精确结清：320 KB/token × 200K，超过 TP=4 后整个块池 60%；MHA 时代 ~524 GB 不可行）。新增练习 2.5（量级估算）与附录 B 答案；ch8 §8.6 补前向指引。

### ch3 · 注意力（第 4 轮）
- **论文验证**：$\text{softmax}(QK^\top/\sqrt{d_k})V$ 与 Vaswani 2017 §3.2.1 一致；缩放理由（点积方差随 $d_k$ 线性增长、推进 softmax 饱和区）即原论文脚注 4 的形式化；因果掩码加性 $-\infty$ 实现与原论文 §3.2.3 一致。
- **工程验证**：PyTorch SDPA 的 `is_causal` 语义即下三角可见；HF modeling_llama 用加性大负数掩码，与正文一致；代码锚点的数值稳定 softmax（减最大值）对齐工程实现惯例。
- **教学验证**：借鉴 3B1B"形容词更新名词向量"的语义位移叙事与 Alammar 逐步拆公式的节奏（均重新表达）；Q/K/V 用"会议三重身份"原创类比并声明边界。
- **前沿补充**：注意力汇（attention sink, Xiao et al. ICLR 2024/StreamingLLM，已检索确认）与滑窗注意力（Mistral）入延伸阅读——属"改变可见性集合"的后续演化，不入正文以免稀释主线。
- **缺口填补**：无已识别缺口；补 softmax 饱和的数值示例（V1 只有结论，诊断 #8）。

### ch4 · 多头与 GQA（第 5 轮）
- **论文验证**：MHA 切分（h×d_k=d_model）与 Vaswani 2017 §3.2.2 一致；MQA 出自 Shazeer 2019；GQA 消融趋势（g=8 与 MHA 几乎持平、g=1 可感下降且大模型更明显）与 Ainslie et al. EMNLP 2023 一致——正文采用定性表述而非杜撰百分比；KV 账单公式 2×h×d_k×2B 与 LLaMA-2 70B 参数自洽（32 KB/token/层 → 10.7 GB/4096 上下文，已数值实测）。
- **工程验证**：HF `repeat_kv` 为逻辑 expand 零拷贝，与正文"读同一块显存地址"表述一致；GQA 下 W_K/W_V 形状 (8192, 1024) 与 HF config（num_key_value_heads=8）对账 ✓。
- **教学验证**：延续 ch3 会议类比（"64 位提问者共享一套会议记录"），保持全书类比体系一致。
- **前沿补充**：MLA 短节入正文（缺口兑现）：低秩联合压缩、~90%+ 压缩率（TransMLA/DeepSeek 线，已检索确认）、RoPE 解耦代价；MHA→GQA→MLA 演化线小图。
- **缺口填补**：已识别缺口"MLA 短节"——4.6 节交付；练习 4 埋 RoPE 兼容问题伏笔（ch9 回收）。

### ch5 · FFN（第 5 轮）
- **论文验证**：SwiGLU 公式与 Shazeer 2020 一致（SiLU(xW_gate)⊙(xW_up)·W_down）；d_ff=28672=3.5×8192 与 HF config 对账 ✓（为门控第三矩阵下调倍率的表述与 GLU 论文动机一致）；键值存储视角与 Geva et al. EMNLP 2021 一致；参数账 3×8192×28672≈705M/层、80 层 56.4B≈81%（数值实测）。
- **工程验证**：结构与 HF `LlamaMLP`（gate_proj/up_proj/down_proj）逐一对应；代码锚点键值分解 einsum 验证通过。
- **教学验证**：微缩 FFN（d=2, d_ff=3）手算例兑现诊断 #8"最小可验证示例"，三组输入全部实测核对；"钥匙-值"叙述重新组织，未复用 V1 文本。
- **前沿补充**：MoE 短节入正文（缺口兑现）：Mixtral 8×7B top-2/8（已检索确认）、DeepSeek-V3 671B/37B 激活；"参数容量与算力解耦 + 显存代价"的权衡表述。
- **缺口填补**：已识别缺口"MoE 短节"与"键值视角最小数值例"——5.5 节与 5.3 节交付。

### ch6 · 残差与归一化（第 6 轮）
- **论文验证**：残差公式与 He et al. 2016 一致；RMSNorm 公式与 Zhang & Sennrich 2019 一致（提速数据改用原论文区间 7%~64%，替换 V1 不可溯源的"20~30%"）；Pre/Post-Norm 与 warmup 关系与 Xiong et al. 2020 一致。
- **工程验证**：Pre-Norm 前向顺序（input_layernorm→attn→残差→post_attention_layernorm→mlp→残差）与 HF LlamaDecoderLayer 一致；Final Norm 对应 model.norm。
- **教学验证**："批注 vs 转述"、"校准原料不校准传送带"为原创表达；残差流（residual stream）术语引自 Elhage 2021 并标注。
- **前沿补充**：本章主题稳定（2019 后无范式变化），无需新增；残差流视角连接可解释性前沿（延伸阅读）。
- **缺口填补**：0.98^80 数值实验替代 V1 的"雅可比谱范数"表述（R1 超纲词消除）。

### ch7 · 输出与采样（第 6 轮）
- **论文验证**：top-p/贪心退化与 Holtzman 2020 一致；top-k 出自 Fan 2018；min-p 出自 Nguyen et al. 2024（arXiv 2407，已检索确认）；repetition penalty 机制与 CTRL（Keskar 2019）定义一致。
- **工程验证**：采样参数覆盖 vLLM SamplingParams（temperature/top_p/top_k/min_p/repetition/presence/frequency penalty，已检索确认）；执行顺序（惩罚→温度→截断→采样）对齐 vLLM 采样器；Final Norm/不共享 LM Head 与 HF LLaMA 一致。
- **教学验证**：借鉴 Transformer Explainer 的"温度滑杆"用三档表格重现；"四把手术刀"框架为原创组织。
- **前沿补充**：min-p 入正文（知识缺口"采样参数全景"兑现）。
- **缺口填补**：缺口表 ch7 行核销。

### ch8 · 自回归与 KV Cache（第 7 轮）
- **论文验证**：Prefill/Decode 两阶段与 Pope et al. 2023 一致；复杂度推导 O(n³d)→O(n²d) 为标准结果；缓存合法性论证基于因果掩码（Vaswani 2017 §3.2.3）。
- **工程验证**：代码锚点"增量==全量"7 步 assert 实测通过——KV Cache 精确等价性的可执行证明；账目 1.34 GB/320 KB/token 与 ch4/附录 A 三处对账一致。
- **教学验证**："q 一次性、k/v 传家"为原创表述；"两副面孔"表格对齐读者体感（TTFT/TPOT）。
- **前沿补充**：attention sink 作为"缓存装不下丢谁"的指针入延伸阅读。
- **缺口填补**：无新增缺口；回收 ch3 种子、ch0/ch7 ★题。

### ch9 · 显存战争（第 7 轮）
- **论文验证**：20~40% 利用率、块表/CoW、2~4× 吞吐与 Kwon et al. SOSP 2023 一致；TP 列切/行切与 Shoeybi 2019 一致；MLA-RoPE 解耦与 DeepSeek-V2 论文的 decoupled RoPE 设计一致。
- **工程验证**：block_size=16 为 vLLM 默认值；vLLM V1 近零开销 prefix caching（2025 blog 已确认）写入正文；TP 同步账（2.6 MB/步 ×160 次）数值复核。
- **教学验证**：OS 分页对照表沿用领域公认类比（PagedAttention 论文自身的类比），表述重写。
- **前沿补充**：三条战线表（量化/前缀复用/MLA）覆盖 2024-2026 主战场。
- **缺口填补**：知识缺口"张量并行节"（9.1）兑现；ch4 练习 4 伏笔（MLA-RoPE）回收。

### ch10 · 吞吐引擎（第 8 轮）
- **论文验证**：Roofline/AI 框架出自 Williams 2009；A100 参数（312 TFLOPS FP16 / ~2 TB/s HBM / SRAM ~19 TB/s）与 NVIDIA 规格及 FlashAttention 论文引用一致；Continuous Batching 出自 Orca (OSDI 2022)；投机解码等价性出自 Leviathan 2023。
- **工程验证**：70 ms/14 token/s 下界经第 8 章练习与本章正文双向对账；vLLM V1 重构动机（控制平面开销）与 2025 官方 blog 一致；Chunked Prefill 为 vLLM/主流引擎现役特性。
- **教学验证**："三味药 + 番外药方"框架、"精确等价工程奇迹收集"（KV Cache/FlashAttention/投机解码三连）为原创组织。
- **前沿补充**：投机解码正文节（缺口兑现）+ DeepSeek MTP 一笔 + FlashAttention-3 延伸阅读。
- **缺口填补**：缺口表 ch10 两行（投机解码、vLLM V1 演进）核销。

### 附录 A/B/C（第 8~9 轮）
- **附录 A**：全部数值与各章正文及脚本复核值一致（参数量合计 ~69B、KV 账本、平衡点 156）；术语表 34 条按 V2 章序标注首现；Mamba/RWKV 指针（知识缺口核销）。
- **附录 B**：全部 42 道练习给出解答/要点；数值题（1.3/4.1/4.2/5.1/5.3/6.1/6.2/8.2/9.1/9.4/10.1）经脚本或手工复核；★ 题答案与正文回收点互相印证。
- **附录 C**：ViT 196 token/16×16 patch 与 Dosovitskiy 2021 一致（已检索确认）；LLaVA 三件套范式与 Liu 2023 一致；复用清单把多模态锚回全书各章（检查点 A 需求核销）。

### 附录 D · 思考的解剖（第 11 轮，用户需求新增）
- **论文验证**：CoT 出自 Wei et al. NeurIPS 2022、零样本"step by step"出自 Kojima et al. 2022；R1 的 RL-可验证奖励路线与思考长度自发增长与 DeepSeek-R1 论文（2025）一致（已检索确认）；"长度换深度优于堆参数"的定量结论与 Snell et al. 2024 一致（已检索确认）。
- **工程验证**：<think> 标记为词表普通 token、思考段按输出价计费与 o1 System Card 及各家 API 文档口径一致；思考预算旋钮对应 OpenAI reasoning effort / Anthropic thinking budget 类参数；思考题 D.1 账目（1.12 GB / 3.6 分钟）按附录 A.3 公式复核。
- **教学验证**："没有新器官——站点⑥的刻意延长"为全书解剖框架的原创延伸；"KV Cache = 工作记忆""用生成实现控制流"为原创表述；两个解释均只用书内已有概念（80 层固定深度、条件分布、掩码可见性）搭建。
- **前沿补充**：覆盖 2022（CoT）→2024（o1/test-time scaling）→2025（R1 开源配方）完整脉络；训练方法（奖励设计/策略优化）显式声明为边界外。
- **缺口填补**：内部素材完全未覆盖本主题——全部内容基于外部权威来源构建；结构落点论证见 AUDIT-LOG 第 11 轮。
