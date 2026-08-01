# 附录 A · 速查手册

> 服务非线性阅读：查形状、查参数、查术语、查出处。所有数值基于全书基准模型 **LLaMA-2 70B**（GQA-8，FP16，与 Meta 官方配置一致：d=8192, L=80, h=64, g=8, d_ff=28672, V=32000）。

---

## A.1 维度全程速查表

一次前向（$n = 10$）从进模型到出模型的完整形状轨迹：

| 阶段 | 输入形状 | 输出形状 | 出处章节 |
|------|----------|----------|---------|
| 分词 | 字符串 | $(n,)$ 整数 | 第 1 章·分词与嵌入 |
| 嵌入查表 | $(n,)$ | $(n, 8192)$ | 第 1 章·分词与嵌入 |
| RMSNorm（子层入口） | $(n, 8192)$ | $(n, 8192)$ | 第 6 章·残差与归一化 |
| Q 投影 + reshape | $(n, 8192)$ | $(n, 64, 128)$ | 第 4 章·多头与 GQA |
| K/V 投影（GQA）+ reshape | $(n, 8192)$ | $(n, 8, 128)$ | 第 4 章·多头与 GQA |
| RoPE 旋转 Q/K | $(n, *, 128)$ | 同形状 | 第 2 章·位置编码 |
| 注意力分数 + 因果掩码 | $Q, K$ | 每头 $(n, n)$，共 64 头 | 第 3 章·注意力 |
| 加权求和 | 分数 × $V$ | $(n, 64, 128)$ | 第 3 章·注意力 |
| 拼接 + $W_O$ 融合 | $(n, 8192)$ | $(n, 8192)$ | 第 4 章·多头与 GQA |
| 残差加法 | $(n,8192)+(n,8192)$ | $(n, 8192)$ | 第 6 章·残差与归一化 |
| FFN gate/up | $(n, 8192)$ | $(n, 28672)$ ×2 支路 | 第 5 章·FFN |
| FFN down | $(n, 28672)$ | $(n, 8192)$ | 第 5 章·FFN |
| ×80 层重复 | $(n, 8192)$ | $(n, 8192)$ | 第 6 章·残差与归一化 |
| Final Norm + 取末行 | $(n, 8192)$ | $(1, 8192)$ | 第 7 章·输出与采样 |
| LM Head | $(1, 8192)$ | $(1, 32000)$ | 第 7 章·输出与采样 |
| 惩罚/温度/截断/采样 | $(1, 32000)$ | 1 个 token id | 第 7 章·输出与采样 |

## A.2 参数量账本

| 部件 | 形状 | 参数量 | 占比 |
|------|------|--------|------|
| 嵌入 $E$ | $(32000, 8192)$ | 0.26 B | ~0.4% |
| $W_Q$ / 层 | $(8192, 8192)$ | 67.1 M | |
| $W_K, W_V$ / 层（GQA） | $(8192, 1024)$ ×2 | 16.8 M | |
| $W_O$ / 层 | $(8192, 8192)$ | 67.1 M | |
| 注意力小计 / 层 | | ~151 M | 80 层共 ~12.1 B（~17%） |
| FFN 三矩阵 / 层 | $(8192, 28672)$ ×2 + $(28672, 8192)$ | ~705 M | 80 层共 ~56.4 B（~81%） |
| RMSNorm $\gamma$ / 层 ×2 + Final Norm | $(8192,)$ | ~1.3 M（全模型） | 忽略不计 |
| LM Head | $(8192, 32000)$ | 0.26 B | ~0.4% |
| **合计** | | **~69 B** | |

两个值得长期记住的比例：**FFN 占八成**（知识库是最大部件，第 5 章）；**GQA 让 K/V 投影只有 Q 的 1/8**（第 4 章）。

## A.3 显存与延迟账本

| 项目 | 公式 | 数值 |
|------|------|------|
| 权重（FP16） | 参数量 × 2 B | ~140 GB（需 TP ≥ 2，常用 4/8） |
| TP=4 每卡权重 | 140 / 4 | ~35 GB |
| KV Cache / token / 层（GQA-8） | $2 \times 8 \times 128 \times 2$ B | 4 KB |
| KV Cache / token（全 80 层） | 4 KB × 80 | 320 KB |
| KV Cache / 请求（4096 上下文） | 320 KB × 4096 | ~1.34 GB |
| 同上若为 MHA | ×8 | ~10.7 GB |
| 同上若为 32K 上下文（GQA-8） | 320 KB × 32768 | ~10.7 GB |
| 单步 Decode 时间下界 | 权重字节 ÷ HBM 带宽 | 140 GB ÷ 2 TB/s = 70 ms |
| 单请求 Decode 速度上限 | 1 ÷ 70 ms | ~14 token/s |
| A100 算术强度平衡点 | 312 TFLOPS ÷ 2 TB/s | 156 |

## A.4 术语表（按首次出现排序）

| 术语 | 一句话定义 | 首现 |
|------|-----------|------|
| token | 分词后的最小文本单元，对应词表一个编号 | 第 0 章 |
| 词表（vocabulary） | 全部候选 token 的清单，大小 $V = 32000$ | 第 0 章 |
| 自回归（autoregressive） | 生成的 token 喂回输入再生成下一个 | 第 0 章 |
| BPE | 反复合并高频相邻对的子词分词算法 | 第 1 章 |
| 字节回退（byte fallback） | 词表外字符退化为 UTF-8 字节 token，永无未知词 | 第 1 章 |
| 嵌入（embedding） | token 编号 → 语义向量的查表 | 第 1 章 |
| 分布式表示 | 语义弥散在众多维度上，单维无独立含义 | 第 1 章 |
| 置换等变性 | 输入行序置换，输出只跟着同样置换 | 第 2 章 |
| RoPE | 按位置成正比的角度旋转 Q/K 的位置编码 | 第 2 章 |
| Q / K / V | 疑问 / 桌牌 / 发言——注意力的三重投影 | 第 3 章 |
| softmax | 指数归一化：分数 → 概率分布 | 第 3 章 |
| 因果掩码 | 下三角 $-\infty$ 常量矩阵，禁止看未来 | 第 3 章 |
| MHA / MQA / GQA | KV 组数 = 64 / 1 / 8 的三代注意力 | 第 4 章 |
| MLA | 把 KV 联合压缩成低秩潜向量缓存 | 第 4 章 |
| SwiGLU | 内容支路 × 门控支路的 FFN 变体 | 第 5 章 |
| 键值存储视角 | FFN = 钥匙检测模式 + 值注入内容 | 第 5 章 |
| MoE | 多专家 FFN + 路由：参数容量与算力解耦 | 第 5 章 |
| 残差连接 / 残差流 | $x + f(x)$；贯穿 80 层的主干道 | 第 6 章 |
| RMSNorm | 只除均方根的轻量归一化 | 第 6 章 |
| Pre-Norm / Final Norm | 校准放子层入口；出口一次总校准 | 第 6/7 章 |
| logits | LM Head 输出的 $V$ 维未归一化分数 | 第 7 章 |
| temperature / top-k / top-p / min-p | 调对比度 / 定额截断 / 累积截断 / 锚定冠军截断 | 第 7 章 |
| repetition / frequency / presence penalty | 三种压制复读的 logits 手术 | 第 7 章 |
| KV Cache | 缓存历史 K/V 免重算；合法性来自因果掩码 | 第 8 章 |
| Prefill / Decode | 啃 prompt（算力瓶颈）/ 逐字生成（带宽瓶颈） | 第 8 章 |
| TTFT / TPOT | 首字延迟 / 字间间隔 | 第 8 章 |
| 张量并行（TP） | 按头/按槽位切矩阵分卡，AllReduce 拼合 | 第 9 章 |
| PagedAttention | KV Cache 分页：块池 + 块表 + CoW | 第 9 章 |
| 内部/外部碎片 | 预留浪费 / 空闲但不连续 | 第 9 章 |
| 前缀缓存 | 跨请求复用相同前缀的 KV 块 | 第 9 章 |
| 算术强度（AI） | FLOPs ÷ 搬运字节；与硬件平衡点比较定瓶颈 | 第 10 章 |
| Continuous Batching | 每步重组批次的调度策略 | 第 10 章 |
| FlashAttention | 分块进 SRAM、不落地 $(n,n)$ 矩阵的精确注意力内核 | 第 10 章 |
| 投机解码 | 小模型猜、大模型并行验，分布严格等价 | 第 10 章 |
| 思维链（CoT） | 提示模型先写推导再给答案——第一代"思考" | 附录 D |
| 思考 token / test-time compute | 用序列长度换串行深度；推理时多花算力换更好答案 | 附录 D |
| 思考预算 | 限制/引导思考段长度的 API 旋钮（reasoning effort 类参数） | 附录 D |

## A.5 符号约定（全书统一）

| 符号 | 含义 | 取值 |
|------|------|------|
| $n$ | 当前序列长度 | prompt 为 10 |
| $V$ | 词表大小 | 32000 |
| $d$（$d_{\text{model}}$） | 隐藏维度 | 8192 |
| $L$ | 层数 | 80 |
| $h$ | Q 头数 | 64 |
| $g$ | KV 组数 | 8 |
| $d_k$ | 每头维度 | 128 |
| $d_{ff}$ | FFN 中间维度 | 28672 |

矩阵形状标注为（行, 列）；向量默认为行向量；对数与指数均为自然底。

## A.6 玩具推理器:把全书代码锚点串起来

全书代码锚点按依赖顺序串联，就是一个能跑通完整流程的微型推理器：

1. 嵌入查表（第 1 章 1.3）→ 2. RoPE 旋转（第 2 章 2.4）→ 3. 单头因果注意力（第 3 章 3.5）→ 4. GQA 分组（第 4 章 4.5）→ 5. SwiGLU FFN（第 5 章 5.4）→ 6. 采样流水线（第 7 章 7.5）→ 7. 带 KV Cache 的增量循环（第 8 章 8.3）。

动手建议：把它们粘进同一个文件、用第 8 章的 assert 做总验收——你会得到一个百余行、纯 NumPy、每一步都与本书公式一一对应的"纸上 vLLM"。

## A.7 延伸阅读总表（按主题）

**架构原典**
- Vaswani et al., *Attention Is All You Need* (NeurIPS 2017)
- Touvron et al., *LLaMA 2* (2023)——基准模型技术报告

**输入与位置**
- Sennrich et al., BPE (ACL 2016)；Kudo & Richardson, *SentencePiece* (EMNLP 2018)
- Su et al., *RoFormer/RoPE* (2021)；Chen et al., 位置插值 (2023)；Peng et al., *YaRN* (2023)；Press et al., *ALiBi* (ICLR 2022)

**注意力及其演化**
- Shazeer, MQA (2019)；Ainslie et al., *GQA* (EMNLP 2023)；DeepSeek-AI, *DeepSeek-V2/V3*（MLA/MTP, 2024）
- Elhage et al., *Transformer Circuits* (2021)；Xiao et al., *Attention Sinks* (ICLR 2024)

**FFN 与知识**
- Shazeer, *GLU Variants* (2020)；Geva et al., *Key-Value Memories* (EMNLP 2021)；Meng et al., *ROME* (NeurIPS 2022)；Jiang et al., *Mixtral of Experts* (2024)

**深度稳定性**
- He et al., *ResNet* (CVPR 2016)；Zhang & Sennrich, *RMSNorm* (NeurIPS 2019)；Xiong et al., Pre-Norm 分析 (ICML 2020)

**解码**
- Holtzman et al., top-p (ICLR 2020)；Fan et al., top-k (ACL 2018)；Nguyen et al., *min-p* (2024)；Press & Wolf, weight tying (2017)

**思考与 test-time compute**
- Wei et al., *Chain-of-Thought Prompting* (NeurIPS 2022)；Kojima et al., *Zero-Shot Reasoners* (NeurIPS 2022)
- DeepSeek-AI, *DeepSeek-R1* (2025)；Snell et al., *Scaling Test-Time Compute* (2024)；OpenAI, *o1 System Card* (2024)

**推理系统**
- Kwon et al., *PagedAttention* (SOSP 2023)；Yu et al., *Orca* (OSDI 2022)；Zheng et al., *SGLang/RadixAttention* (2024)
- Dao et al., *FlashAttention* (NeurIPS 2022)；Shoeybi et al., *Megatron-LM* (2019)
- Pope et al., *Efficiently Scaling Transformer Inference* (MLSys 2023)；Leviathan et al., 投机解码 (ICML 2023)
- Williams et al., *Roofline* (CACM 2009)

**注意力 O(n²) 之外的出路**（本书未展开，给一个方向指针）
- Gu & Dao, *Mamba: Linear-Time Sequence Modeling with Selective State Spaces* (2023)；Peng et al., *RWKV* (2023)——状态空间/线性递归路线：常数级"记忆"替代随长度增长的 KV Cache，是对本书第 8~9 章问题的另一种回答

---

⬅️ 上一站：[第 10 章 · 吞吐引擎](./part4-chapter10-吞吐引擎.md) ｜ [返回目录](./README.md) ｜ ➡️ [附录 B · 练习提示与答案](./appendix-b-练习答案.md)
