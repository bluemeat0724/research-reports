# 企业级本地化（私有化/On-Premises）大模型部署研究报告
## —— 面向数据安全的模型、硬件、推理框架、合规与参考架构全景

- **报告日期**：2026-10-14（亚洲/上海，UTC+8）
- **适用对象**：需要「数据不出域」的政企、金融、医疗、能源、制造、科研等组织的架构师 / 安全负责人 / 采购决策者
- **方法说明**：本报告所有关键事实均附来源链接，且**仅引用本次实际抓取（WebFetch/WebSearch 返回）的页面**。
  - 标注 `【估算】` 的数字为本报告基于公开参数做的推导，不是厂商口径。
  - 标注 `【未核实】` 的为搜索结果中出现、但本次未逐页核验原始文档的内容，**采购决策前必须自行核验**。
  - 价格类数据波动极大，均为**某一时点报价**，仅作量级参考。

---

## 0. 执行摘要（TL;DR）

1. **模型侧已不是瓶颈**。2026 年开放权重（open-weight）模型在编码、Agent、长上下文上已逼近甚至部分超过闭源旗舰：DeepSeek-V4-Pro-0813（1.6T/49B 激活，MIT）在 Terminal Bench 2.1 达 87.9、GLM-5.3（744B/40B，GLM-5.3 License）在 GDPval-AA v2 达 1769、Kimi K3（2.8T/104B）与 Qwen3.8-Max（2.4T/95B）同处第一梯队。真正的工程难点转移到**显存、互联、PD 分离与合规**。
2. **许可证差异比参数更关键**。DeepSeek 全系与 gpt-oss、Mistral Large 3、Gemma 4 是宽松许可（MIT / Apache-2.0）；GLM-5.3、Qwen3.8-Max、Kimi K3 属「附条件商用」；MiniMax-M3 的 MINIMAX COMMUNITY LICENSE **默认仅授予非商业用途**，商用需署名+通知，年收入 >2000 万美元须事先书面授权。企业内部自用通常不触发大部分条款，但法务必须逐条过。
3. **中国境内合规有一个关键豁免**。《生成式人工智能服务管理暂行办法》第二条明确：企业、科研机构研发应用生成式 AI 技术但**未向境内公众提供服务的，不适用本办法**——纯内部私有化部署一般不需要大模型备案；一旦对外（含 2C/2B 公众可及）提供，则需安全评估+算法备案（第十七条），并受《AI 生成合成内容标识办法》与 GB 45438—2025 强制性国标约束。
4. **硬件呈「三轨并行」**。①受限进口（H20 96GB、H200 141GB，经 NDRC 逐单审批、量极小）；②国产加速卡（昇腾 910B/910C、950PR/950DT、寒武纪 MLU690、昆仑芯 P800、沐曦 C600、摩尔线程 S5000、海光 BW1000）；③消费/工作站卡（RTX 5090 32GB、RTX PRO 6000 96GB）用于中小规模与边缘。国产卡 2026 年在推理侧已可规模落地，训练侧仍有差距。
5. **推理栈的分水岭是 PD 分离 + KV 复用**。vLLM v0.31.0 / SGLang 均已把 Prefill-Decode 分离产品化（NIXL / Mooncake / LMCache / FlexKV 连接器），实测 goodput 提升 2.4×、p99 ITL 从 263ms 降到 ≤52ms；但 KV 传输慢时 TTFT 反而恶化——**没有 RDMA/NVLink 就不要做 PD 分离**。
6. **量化已是「免费午餐」级成熟**。FP8（W8A8-FP）基本无损（恢复率 ~99.7–100%），INT4 W4A16 平均恢复 99.36%，NVFP4 在多数任务与 BF16 差距 ≤1%。但小模型（1.5B 级）INT4 会掉到 93.5% 恢复率——**模型越小越不能激进量化**。
7. **网关层必须自建**。LiteLLM（虚拟密钥/预算/护栏）+ Higress（Token 限流/内容安全）+ Open WebUI（SSO/RBAC/审计，免费版即含）构成「可审计的私有 AI 入口」，是等保与审计取证的落点。
8. **RAG 是私有化最大价值场景**，也是提示注入（OWASP LLM01:2025 间接注入）的主要入口，必须按「不可信内容」隔离处理。

---

## 1. 模型层：主流开放权重模型全景（2026-10 口径）

### 1.1 版本核验结果（含「已过时版本」提示）

| 家族 | 当前主力开放权重版本 | 发布时间 | 总参数 / 激活参数 | 上下文 | 原生精度 | 许可证 | 来源 |
|---|---|---|---|---|---|---|---|
| DeepSeek | **DeepSeek-V4-Pro-0813**（正式版，含 DSpark 投机解码模块） | 2026-08-13 | 1.6T / 49B | 1M | FP4+FP8 混合（MoE 专家 FP4） | MIT | [HF README](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813/raw/main/README.md)、[V4 预览公告](https://www.deepseek.com/en/news/v4-preview/) |
| DeepSeek | DeepSeek-V4-Flash（经济型） | 2026-04-24（预览）/ 0731 更新 | 284B / 13B | 1M | FP4+FP8 混合 | MIT | [HF Flash](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)、[Pro-0813 对比表](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813/raw/main/README.md) |
| DeepSeek | DeepSeek-V3.2（上一代，仍在用） | 2025 | 685B / 37B | — | FP8 | MIT | [HF V3.2](https://huggingface.co/deepseek-ai/DeepSeek-V3.2) |
| Qwen | **Qwen3.8-2.4T-A95B**（Max 级首次开放权重，纯文本） | 2026-08-12 | 2.4T / 95B | 262,144 原生，可扩至 1,010,000 | （见模型卡） | Qwen3.8-Max License（>1 亿 MAU 或月收入 >2000 万美元须标注模型名；MaaS/AI 工作助手业务且 12 个月收入 >5000 万美元须另取许可；**内部使用豁免**） | [HF README](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md)、[LICENSE](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/LICENSE) |
| Qwen | **Qwen3.8-27B**（稠密，原生多模态 VL） | 2026-08-14 | 27B 稠密 | 262,144 → 1M（YaRN） | BF16 | **Apache-2.0** | [HF README](https://huggingface.co/Qwen/Qwen3.8-27B/raw/main/README.md) |
| Qwen | Qwen3.8-Flash-Next（Qwen4 架构预览，多模态 MoE） | 2026-08-26 | 125B 主模型 + 51B N-gram 嵌入 / 6B 激活 | — | — | 【未核实】 | 搜索结果摘要（GitHub qwenlm/qwen3.8-flash-next，原页未抓取）【未核实】；X 用户 @yibie 称 strata 引擎可在 12GB 显卡运行该 125B MoE（[X 帖](https://x.com/bruc3van/status/2109149893390418263) 引述，个人经验） |
| Qwen | Qwen3.6-35B-A3B（性价比主力，VL） | 2026-04-16 | 35B / 3B（256 专家，8 路由+1 共享） | 262,144 → 1,010,000 | BF16 | **Apache-2.0** | [HF README](https://huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/README.md) |
| Zhipu / Z.AI | **GLM-5.3**（后训练升级，与 5.2 同基座） | 2026（arXiv 2602.15763） | 744B / 40B（256 专家，80 层，DSA 稀疏注意力） | — | FP8 / BF16 | GLM-5.3 License（类 MIT；仅当运营 MaaS 且集团 12 个月收入 >100 亿美元时须通过 Z.AI 安全审查） | [HF README](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md)、[LICENSE](https://huggingface.co/zai-org/GLM-5.3/raw/main/LICENSE) |
| Zhipu / Z.AI | **GLM-5.3-Flash**（首个原生多模态，混合稀疏+线性注意力，mHC） | 2026 | 320B / 18B | — | FP8 / BF16 | **MIT**（模型卡 license 字段） | [HF README](https://huggingface.co/zai-org/GLM-5.3-Flash/raw/main/README.md) |
| Moonshot | **Kimi K3**（世界首个开放 3T 级，KDA+AttnRes，原生视觉） | 2026-07（LICENSE 提交 3 个月前） | 2.8T / 104B（896 专家选 16，93 层） | 1,048,576 | **MXFP4 权重 / MXFP8 激活（QAT）** | Kimi K3 License（MaaS 且 12 个月收入 >2000 万美元须另签协议；>1 亿 MAU 或月收入 >2000 万美元须显著标注；**内部使用明确豁免**） | [HF README](https://huggingface.co/moonshotai/Kimi-K3/raw/main/README.md)、[LICENSE](https://huggingface.co/moonshotai/Kimi-K3/blob/main/LICENSE) |
| Moonshot | Kimi K2.5（上一代，原生多模态） | 2026-01-26 | 1T / 32B（384 专家选 8，61 层，MLA） | 256K | 原生 INT4 weight-only（group 32，面向 Hopper） | Modified MIT（>1 亿 MAU 或 >2000 万美元月收入须显著标注 "Kimi K2.5"） | [HF README](https://huggingface.co/moonshotai/Kimi-K2.5/raw/main/README.md)、[LICENSE](https://huggingface.co/moonshotai/Kimi-K2.5/blob/main/LICENSE) |
| MiniMax | **MiniMax-M3**（原生多模态，MSA 稀疏注意力） | 2026-06-12 | ~428B / ~23B | 1M | — | **MINIMAX COMMUNITY LICENSE（默认仅非商用）** | [HF README](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/README.md)、[LICENSE](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/LICENSE) |
| MiniMax | MiniMax-M2.5 | 2026-02-12 | 229.9B / 9.8B（HF 显示 229B） | 192K 原生（max_position_embeddings 196608） | FP8（E4M3，block 128×128） | MIT【未核实：以 HF LICENSE 为准】 | [HF](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)、[InferenceX](https://inferencex.semianalysis.com/model/minimax-m27) |
| OpenAI | **gpt-oss-120b / gpt-oss-20b**（仍是唯一通用开放权重，无 gpt-oss-2） | 2025-08 | 117B / 5.1B；21B / 3.6B | — | **MXFP4（MoE 权重）** | **Apache-2.0** | [HF gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b) |
| Mistral | **Mistral Large 3（675B-A41B）** | 2025-12-02 | 675B / 41B | 256K | BF16 + 官方 NVFP4 检查点 | **Apache-2.0** | [Mistral 3 公告](https://mistral.ai/news/mistral-3/) |
| Mistral | Ministral 3（3B / 8B / 14B，含 base/instruct/reasoning，均带视觉） | 2025-12-02 | 稠密 | — | — | **Apache-2.0** | [同上](https://mistral.ai/news/mistral-3/) |
| Mistral | Mistral Large 4（ML4）：**权重尚未发布** | 2026-10-06 预览 API | 1T / 52B | — | — | 承诺开放权重，月底发布 | [ML4 公告](https://mistral.ai/news/mistral-large-4/) |
| Google | **Gemma 4**（E2B / E4B / 12B Unified / 26B-A4B MoE / 31B Dense） | 2026-03-31（12B 于 06-03） | 31B 稠密；26B 总 / 3.8B 激活（128 专家选 8 + 1 共享） | 小模型 128K，中型 256K | — | **Apache-2.0** | [Gemma 4 模型卡](https://ai.google.dev/gemma/docs/core/model_card_4) |
| Meta | **Llama 4 Scout / Maverick（仍是最新可下载代际，未见 Llama 5 权重）** | 2025-04-05 | Scout 109B/17B（16 专家）；Maverick 400B/17B（128 专家） | Scout 10M；Maverick 1M | BF16；Maverick 另有 FP8 | Llama 4 Community License（>7 亿 MAU 需 Meta 批准；欧盟主体多模态权利受限） | [Llama 4 Model Card](https://raw.githubusercontent.com/meta-llama/llama-models/main/models/llama4/MODEL_CARD.md) |

> ⚠️ **选型提示**：若你的资料/方案里还写着「DeepSeek-R1 671B」「Qwen2.5-72B」「Llama 3.1 405B」作为 2026 年的主力选项，请更新——这些仍可用，但已非当前代际。

### 1.2 关键基准（含中文能力）

**（a）旗舰横评（GLM-5.3 官方表，口径与 harness 见原文脚注）** — 来源：[GLM-5.3 模型卡](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md)

| Benchmark | GLM-5.3 | GLM-5.2 | Kimi K3 | DeepSeek-V4-Pro-0813 | Qwen3.8-Max | Opus 4.8 | GPT-5.6 Sol |
|---|---|---|---|---|---|---|---|
| Terminal Bench 2.1 | 88.2 | 81.0 | 88.3 | 87.9 | 86.6 | 85.0 | **88.8** |
| Terminal Bench 3.0 | 28.3 | 4.6 | 17.4 | – | – | 21.1 | **34.6** |
| DeepSWE v1.1 | 66.9 | 46.2 | 67.5 | 62.7 | 56.6 | 58.0 | **72.7** |
| CyberGym（漏洞发现） | **84.5** | 77.2 | 80.0 | 83.3 | 78.5 | 78.1 | 83.6 |
| Toolathlon Verified | 73.0 | 59.9 | **76.5** | 74.1 | 72.5 | 76.2 | 74.9 |
| AutomationBench v1.0.6 | **48.2** | 26.2 | 46.7 | 43.2 | 39.8 | 41.0 | 45.8 |
| HLE w/ Tools | 62.5 | 54.7 | 59.8 | 60.0 | 56.2 | 57.9 | **64.5** |
| GDPval-AA v2（Elo） | **1769** | 1508 | 1682 | 1590 | 1739 | 1588 | 1730 |

**（b）DeepSeek 自家对比** — 来源：[DeepSeek-V4-Pro-0813](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813/raw/main/README.md)：V4-Pro-0813 的 HLE（无工具/有工具）42.7/60.0、NL2Repo 61.5、DSBench-Hard 67.2，显著高于 V4-Pro 预览版（37.7/48.2、38.5、31.1）——**「预览版」与「0813 正式版」性能差距很大，部署务必用 0813。**

**（c）中文能力**：当前头部模型的厂商卡多已不再单列 C-Eval/CMMLU。可核验的中文基准：
- Qwen3.6-35B-A3B 的 **C-Eval 90.0**，Qwen3.5-27B 90.5，Gemma4-31B 82.6，Gemma4-26B-A4B 82.5 —— 来源：[Qwen3.6-35B-A3B 模型卡](https://huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/README.md)。即**同量级下国产模型中文领先 Gemma 约 8 分**。
- Qwen3.8-Max 列有 **PLawBench 73.2**（中文法律，gemini-3.1-pro 评审）、PRBench-Finance 58.3、HealthBench 60.2，均为同表最高 —— 来源：[Qwen3.8-2.4T-A95B 模型卡](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md)。
- GLM 系列模型卡语言字段为 `en, zh`，Kimi 词表 160K、Qwen 词表 248,320（padded），中文分词效率较 Llama/Gemma（262K 但英文主导）更优 —— 来源同各模型卡。
- Gemma 4 预训练覆盖 140+ 语言、开箱支持 35+ 语言，但其音频 CoVoST/FLEURS 指标**明确标注「不含中文」** —— 来源：[Gemma 4 模型卡](https://ai.google.dev/gemma/docs/core/model_card_4)。
- **结论**：中文 + 中国法律/金融/政务语境下，Qwen / GLM / DeepSeek / Kimi 四家是默认候选；Llama / Gemma / Mistral 仅在多语种或 Apache 许可硬约束时考虑。

### 1.3 量化格式与质量影响

**格式总览**

| 格式 | 权重/激活 | 典型工具 | 硬件要求 | 适用场景 |
|---|---|---|---|---|
| BF16 | 16/16 | 原生 | 任意 | 基线、精度敏感评测 |
| FP8（W8A8-FP，E4M3） | 8/8 | llm-compressor、原生 FP8 权重 | Hopper / Ada / Blackwell；**海光 BW1000 明确不支持 FP8** | 高吞吐连续批处理首选 |
| INT8（W8A8-INT） | 8/8 | GPTQ + SmoothQuant / 动态 per-token | Ampere 及更老、多数国产卡 | 老卡 / 国产卡主力 |
| INT4 AWQ / GPTQ（W4A16） | 4/16 | llm-compressor、AutoAWQ | 广泛 | 低并发低延迟、显存受限 |
| GGUF（llama.cpp / Ollama） | 2–8 bit 多档 | llama.cpp | CPU/消费 GPU/Apple | 单机、边缘、PoC |
| MXFP4 / MXFP8 | 4 或 8，microscaling | 原生（gpt-oss、Kimi K3 QAT） | **Hopper 及以后（CC ≥ 7.5）+ Triton ≥ 3.4** | 旗舰模型原生格式 |
| NVFP4 | 4/4 或 4/16 | NVIDIA Model Optimizer | Blackwell | Blackwell 上的极致吞吐 |

**质量影响（可引用的硬数据）**

来源：Kurtić et al., *"Give Me BF16 or Give Me Death"? Accuracy-Performance Trade-Offs in LLM Quantization*（Red Hat AI + ISTA，50 万+ 次评测，Llama-3.1 全系）— [arXiv:2411.02355v4](https://arxiv.org/html/2411.02355v4)

| 格式 | Open LLM Leaderboard V1 平均恢复率 | 结论 |
|---|---|---|
| W8A8-FP（FP8） | **99.75%**（405B 甚至 100.12%） | **事实无损** |
| W8A8-INT | 平均每任务仅降 1–3%；405B 在 LB V2 为 98.3% | 需 SmoothQuant + 高质量校准集 |
| W4A16-INT（GPTQ） | **99.36%** | 与 8-bit 同级，优于预期 |

- **GPTQ vs AWQ**：学术榜几乎打平（AWQ 领先 0.23/0.35 分），但**真实任务 GPTQ 领先 2.9 / 0.8 分**（Arena-Hard、HumanEval、MBPP）。论文推荐 GPTQ + MSE 最优 clipping + group_size=128 + OpenPlatypus 校准。
- **小模型风险**：DeepSeek-R1-Distill-Qwen-1.5B 的 W4A16 恢复率仅 **93.5%**（AIME24 从 30.1 掉到 24.6）；7B 为 98.3%，32B 为 99.5%。→ **<7B 的模型不要上 INT4。**
- **部署取向**：W4A16 在**同步/低延迟**场景最省（405B 上成本降 5–7×，4 卡即可替代 BF16 的 16 卡）；W8A8 在**异步连续批处理**吞吐最高。
- **长上下文**：RULER（4k–128k）所有格式恢复率 ≥98%。
- **NVFP4**：NVIDIA 称在标准任务上通常与 BF16 差距 ~1% 以内，DeepSeek-R1-0528 相对 FP8 劣化 ≤1%；但 W4A4 NVFP4 PTQ 在 IFBench 掉 2.6 分，需量化感知蒸馏（QAD）才能追平，MMMU-Pro 仍留 0.7 分差距 —— 来源：[NVIDIA 技术博客](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)；其中 IFBench −2.6 分 / QAD 追平 / MMMU-Pro 0.7 分差距来自 NVIDIA Model Optimizer QAD 公告的搜索摘要（原页本次未抓取 →【未核实】）。
- **原生量化是新趋势**：Kimi K3 自 SFT 阶段起就做 QAT（MXFP4 权重 / MXFP8 激活），DeepSeek-V4 的 MoE 专家权重原生 FP4，gpt-oss 的全部评测都在 MXFP4 下完成。→ **对这些模型，"原生格式" 就是基线，不应再自行二次量化。**

**量化落地建议**
1. 先跑 BF16/原生格式建立基线 → 用 `lm-eval` 经 vLLM 评测 → 再逐级量化，**在你自己的业务集上**算恢复率。
2. 企业内常见最优解：**MoE 旗舰用原生 FP8/FP4；30B 级稠密用 FP8；14B 以下保持 BF16 或 FP8**。
3. KV cache 量化（fp8 kv）可显著扩并发（DeepSeek-V4 官方 vLLM 命令即用 `--kv-cache-dtype fp8`），但属于论文「未覆盖」的部分，需单独验证长上下文检索能力。

---

## 2. 硬件层：GPU / 国产加速卡 / 一体机

### 2.1 NVIDIA 数据中心与工作站卡

| 型号 | 显存 | 带宽 | FP8/FP4 算力 | 互联 | 功耗 | 中国可得性 | 来源 |
|---|---|---|---|---|---|---|---|
| **H20（HGX）** | 96 GB HBM3 | 4.0 TB/s | FP8 296 TFLOPS；无 FP4 | NVLink 900 GB/s | 400W | 受限出口专供，需逐单审批 | [Tom's Hardware 规格表](https://www.tomshardware.com/pc-components/gpus/new-nvidia-ai-gpus-designed-to-get-around-us-export-bans-come-to-china-h20-l20-and-l2-to-fill-void-left-by-restricted-models) |
| **L20 PCIe** | 48 GB GDDR6 ECC | 864 GB/s | FP8 239 TFLOPS | 无 NVLink | 275W | 同上 | 同上 |
| **L2 PCIe** | 24 GB GDDR6 ECC | 300 GB/s | FP8 193 TFLOPS | 无 | — | 同上 | 同上 |
| **H200 SXM** | 141 GB HBM3e | 4.8 TB/s | — | NVLink | 700W | 2026 年起经中方「逐单进口许可」，首批获批给字节、腾讯 | [NVIDIA H200](https://www.nvidia.com/en-us/data-center/h200/)、[Tom's Hardware：中国批准首批 H200](https://www.tomshardware.com/pc-components/gpus/china-approves-first-nvidia-h200-deliveries-to-bytedance-and-tencent-under-case-by-case-import-licenses) |
| **HGX B200（8×）** | 1.4 TB 总 | — | FP4 144/72 PFLOPS（稀疏/稠密）、FP8 72 PFLOPS | NVLink5 1.8 TB/s/GPU，总 14.4 TB/s | — | 对华未开放 | [NVIDIA HGX 规格](https://www.nvidia.com/en-us/data-center/hgx/) |
| **HGX B300（8×）** | 2.1 TB 总 | — | FP4 144/108 PFLOPS、FP8 72 PFLOPS | 同上 | — | 对华未开放 | 同上 |
| **GB300 NVL72 / 单卡** | 单卡约 **279 GB 可用**（vLLM 实测 276.62 GiB 可分配） | — | — | NVLink72 | — | 对华未开放 | [vLLM PD 博客（GB300 实测内存账）](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving) |
| **HGX (Vera) Rubin NVL8** | 2 TB HBM4 \| 130 TB/s | 单卡 288 GB HBM4 \| 22 TB/s | NVFP4 推理 400 PFLOPS | NVLink6 24 TB/s | — | 已量产出货（2026），对华未开放 | [NVIDIA HGX](https://www.nvidia.com/en-us/data-center/hgx/) |
| **RTX PRO 6000 Blackwell Server Edition** | **96 GB GDDR7 ECC** | 1597 GB/s | FP4 4 PFLOPS、FP8 2 PFLOPS、BF16 1 PFLOPS | 无 NVLink（PCIe Gen5）、支持 4 路 MIG | ≤600W 可调 | 相对易得 | [NVIDIA 官方规格](https://www.nvidia.com/en-us/data-center/rtx-pro-6000-blackwell-server-edition.md) |
| **RTX 5090** | 32 GB GDDR7 | — | — | 无 | — | 消费级 | [NVIDIA RTX 5090](https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/rtx-5090/) |

> **关键选型结论**：
> - **无 NVLink 的卡（L20/L40S/RTX PRO 6000/4090/5090）不适合跑需要 TP≥4 的大 MoE**，PCIe all-reduce 会成为瓶颈；它们适合 ≤32B 稠密模型、单卡/双卡部署、以及 Embedding/Rerank/ASR 等配套模型。
> - RTX PRO 6000 Server Edition 的 96 GB + 1.6 TB/s + 4 路 MIG，是目前**企业数据中心「一卡多租」最务实的选择**；MIG 隔离同时也是等保「资源隔离」的技术抓手。
> - 消费级 RTX 4090/5090 用于生产存在**驱动授权（NVIDIA GeForce 驱动的数据中心使用限制）、无 ECC、无官方服务器整机质保**三重风险 →【需法务与采购评估】。

### 2.2 国产加速卡

| 厂商 / 型号 | 显存 | 带宽 | 精度支持 | 互联 | 上市/路标 | 来源 |
|---|---|---|---|---|---|---|
| 昇腾 **910B1/B2/B3** | 64 GB HBM2e | 392 GB/s【未核实：该数值与同表 910A 的 1.5 TB/s 自相矛盾，疑为笔误，务必以华为官方 datasheet 为准】 | INT8 828/752/626 T，FP16 414/376/313 T；**无 FP8/FP4** | 200 Gbps | 2021–2022 | [AI柠檬《面向AI的华为昇腾NPU参数汇总整理》](https://blog.ailemon.net/2025/05/24/huawei-ascend-npu-params-for-ai/)（该站声明数据来自互联网公开资料，部分为合理推测） |
| 昇腾 **910C** | 128 GB HBM3 | 784 GB/s | INT8 1600/1504 T，FP16 800/752 T；**无 FP8/FP4** | 3.2 TB/s（400Gbps） | 2025 Q1，Atlas 900 A3 SuperPoD / CloudMatrix384（384 卡 300 PFLOPS） | 同上 + [徐直军 HC2025 演讲](https://www.huawei.com/cn/news/2025/9/hc-xu-keynote-speech) |
| 昇腾 **950PR** | 128 GB（自研 HiBL 1.0 HBM） | 1.6 TB/s | **FP8/MXFP8/MXFP4/HiF8**，FP8 1P、FP4 2P | 2 TB/s | **2026 Q1**，面向 Prefill 与推荐 | [徐直军 HC2025 演讲](https://www.huawei.com/cn/news/2025/9/hc-xu-keynote-speech) |
| 昇腾 **950DT** | 144 GB（HiZQ 2.0） | 4 TB/s | 同上 | 2 TB/s | **2026 Q4**，面向 Decode 与训练 | 同上 |
| 昇腾 **960 / 970** | 288 GB | 9.6 / 14.4 TB/s | 960 增 HiF4 | 2.2 / 4 TB/s | 2027 Q4 / 2028 Q4 | 同上 + AI柠檬汇总 |
| **Atlas 950 SuperPoD** | 8192 卡，1152 TB 内存 | 互联 16.3 PB/s | FP8 8 EFLOPS、FP4 16 EFLOPS | 灵衢 UnifiedBus 2.0 | 2026 Q4 | [徐直军演讲](https://www.huawei.com/cn/news/2025/9/hc-xu-keynote-speech) |
| **Atlas 950 SuperPoD（WAIC 实机）** | 1024 卡部署，256 TB 全局统一内存编址 | RTT 3 μs | FP8 1 EFLOPS、FP4 2 EFLOPS | 同上 | 2026-07 真机亮相 | [华为新闻](https://www.huawei.com/cn/news/2026/7/atlas-950-superpod) |
| **Atlas 850E 风冷超节点** | 单超节点最大 96 卡 | 内存带宽 4.0 TB/s | FP8/mxFP8/HiF8/INT8/mxFP4 | 灵衢 | 2026 | 同上 —— **对无法做液冷改造的传统机房意义重大，标准机柜可直接上架，10ms 级时延** |
| **海光 BW1000（深算）** | 64 GB HBM2e | 1.6 TB/s | FP16 480 T / FP32 60 T / FP64 30 T；**明确不支持 FP8** | XGMI | 在售 | [模力方舟文档](https://moark.com/docs/compute/clusters_gpu/hygon_gpu) |
| **摩尔线程 MTT S5000（平湖）** | 80 GB | 1.6 TB/s | **原生 FP8**（累加尾数 24bit）、FP16/BF16/INT8/FP32/FP64；稠密 1000 TFLOPS | MTLink 2 代，8 卡全互联 784 GB/s | 在售 | [模力方舟文档](https://moark.com/docs/compute/clusters_gpu/moore_gpu) |
| 摩尔线程 MTT S4000 | 48 GB GDDR6 | 768 GB/s | INT8 200T / FP16 100T | — | 上一代 | [AI柠檬国产卡汇总](https://blog.ailemon.net/2025/10/31/national-ai-chip-param-info-collection/) |
| **昆仑芯 P800 OAM** | 96 GB | 【未核实】 | **仅 FP16 / FP32 / INT8**（官方规格表无 FP8） | PCIe Gen5 x16 + XDR/RDMA | 在售 | [腾讯云 P800 部署实践](https://cloud.tencent.com/document/product/1397/127039) |
| 昆仑芯 R480-X8 / R200 等 | 32 GB GDDR6 | 512 GB/s | INT8 256T | 200 GB/s | 上一代 | [AI柠檬汇总](https://blog.ailemon.net/2025/10/31/national-ai-chip-param-info-collection/) |
| **寒武纪 MLU590** | 96 GB HBM2e | 2.76 TB/s | INT8 628T、FP16/BF16 314T（FP8 未公开披露） | — | 在售 | 同上 |
| 寒武纪 MLU370-X8 | 48 GB LPDDR5 | 614.4 GB/s | INT8 256T、FP16 96T | 200 GB/s | 上一代 | 同上 |
| **沐曦 曦云 C600** | **144 GB HBM3e** | 【未核实】 | **支持 FP8**，混合精度 | MetaXLink 超节点扩展 | 2025-10-20 发布，**首款全国产供应链通用 GPU** | [IT之家报道](https://www.ithome.com/0/890/942.htm) |
| 沐曦 曦云 C500 | 64 GB HBM2e | 1.8 TB/s | INT8 560T、FP16 280T | — | 上一代 | [AI柠檬汇总](https://blog.ailemon.net/2025/10/31/national-ai-chip-param-info-collection/) |
| 天数智芯 天垓150 | 64 GB HBM2e | 1.6 TB/s | INT8 384T | — | 在售，参考价 ￥80,759 | 同上 |
| 璧仞 BR100 | 64 GB HBM2e | 2.3 TB/s | INT8 2048T、FP16 1024T | 128 GB/s | — | 同上 |
| 昇腾 310P（Atlas 300I Duo） | 48/96 GB LPDDR4X | 408 GB/s | INT8 280 TOPS、FP16 140 TFLOPS | — | 推理/视频解析卡 | [AI柠檬昇腾汇总](https://blog.ailemon.net/2025/05/24/huawei-ascend-npu-params-for-ai/) |

**国产卡落地的三个硬约束（务必在 POC 阶段验证）**
1. **精度格式**：海光 BW1000 **不支持 FP8，强行加载 HF 上的 FP8 权重会直接报错**（官方文档原话）；昆仑芯 P800 官方规格仅列 FP16/FP32/INT8。而 GLM-5.3、MiniMax、DeepSeek-V4、Kimi K3 的**官方权重就是 FP8/FP4/MXFP4** → 必须使用厂商提供的 **INT8/INT4 重量化版本**（如昆仑芯提供 `MiniMax-M3-int4`、`DeepSeek-R1-Channel-INT8-vllm`）。
2. **软件栈锁版本**：海光 DTK（`torch 2.4.1+das.opt1.dtk25041`、`vllm 0.8.5.post1+das...`）与摩尔线程 MUSA（`vllm 0.16.1rc1...musa`）都是**厂商定制轮子**，一次 `pip install torch` 就会覆盖掉适配版导致整机不可用。CI/CD 必须锁死镜像。
3. **主机侧配置**：昆仑芯 P800 要求 PCIe Gen5 x16、BIOS `MMIO High Granularity Size = 1024G`、容器 `--privileged`、`--shm-size 256g`、OFED 5.8-2.0.3 —— 这类「隐性需求」是国产卡交付延期的最常见原因。

**生态成熟度（按本次可核验证据排序）**：昇腾（CANN 开源 67 个项目、1244 万行代码、Atlas 384 超节点商用 750+ 套，20+ 行业）> 摩尔线程（GLM-5 Day-0 适配、SGLang/vLLM 双栈、Megatron 万卡训练 loss 差 0.62%）≈ 昆仑芯（SGLang PD 分离镜像 + EAGLE3 + MTP 已就绪）> 海光（vLLM 可用但无 FP8）> 其余。

### 2.3 显存需求测算（按模型规模）

权重显存 **【估算】** 公式：`参数量 × 每参数字节数 × 1.05（框架开销）`；BF16=2 B、FP8=1 B、INT4≈0.55 B（含 scale）。KV cache 另算。

| 模型 | 格式 | 权重显存【估算】 | 最小可行配置【估算】 | 备注 |
|---|---|---|---|---|
| Qwen3.8-27B（稠密） | BF16 | ~57 GB | 1×RTX PRO 6000 96GB 或 2×L20 48GB | 长上下文需再留 20–40 GB KV |
| Qwen3.8-27B | FP8 | ~28 GB | 1×L20 48GB / 1×RTX 5090 32GB（紧张） | |
| Qwen3.6-35B-A3B（MoE） | FP8 | ~37 GB | 1×RTX PRO 6000 96GB | 激活仅 3B，解码极快 |
| Gemma4-26B-A4B | BF16 | ~55 GB | 1×96GB 卡 | |
| MiniMax-M2.5 230B/9.8B | FP8 | ~241 GB | 4×H20 96GB（384 GB） | |
| GLM-5.3-Flash 320B/18B | FP8 | ~336 GB | 4×H20 / 8×昇腾910B 64GB | |
| DeepSeek-V4-Flash 284B/13B | FP4+FP8 混合 | ~180–250 GB【估算，混合精度无法精确换算】 | 4×H20 或 8×910B | |
| GLM-5.3 744B/40B | FP8 | ~781 GB | 8×H20 96GB（768 GB，**刚好不够，需 INT4 或 16 卡**） / 8×昇腾950DT 144GB（1152 GB） | |
| Kimi K2.5 1T/32B | 原生 INT4 | ~550 GB | 8×H20 | 官方 group-32 INT4，面向 Hopper |
| DeepSeek-V4-Pro 1.6T/49B | FP4 专家 + FP8 | ~900 GB–1.1 TB【估算】 | 16×H20 / 8×昇腾950DT | |
| Kimi K3 2.8T/104B | MXFP4 | ~1.5 TB【估算】 | ≥16×96GB 或 Atlas 950 超节点 | 需 Hopper 及以后支持 MXFP4 |
| Qwen3.8-2.4T-A95B | NVFP4（MoE）+BF16 | 实测 1242 GiB 专家 + 91 GiB 非专家 = **1333 GiB** | 8×GB300（TP8 每卡 169.28 GiB 权重） | 来源：[vLLM PD 博客](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving) ——**这是本报告唯一一条厂商实测而非估算的权重显存数据** |

**KV cache 的真实开销（必读）**：以 Qwen3.8-2.4T 为例，vLLM 实测单请求 KV 占 **759 MiB**（ISL 8192 / OSL 1024），单引擎可用 KV 空间 83–134 GiB，**最大并发仅 98–181 路**；且该模型因 GDN（线性注意力）state 为 4 MiB/请求（是 Full-Attn 每 token 2 KiB 的 2108 倍），block size 被迫设为 2112 token / 4.125 MiB。另外「既非权重也非 KV」的固定开销约 20 GiB/卡（≈7%）。→ 来源：[vLLM PD 博客](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)。**凡是只按权重算显存的方案书，都低估了 30–50%。**

### 2.4 价格参考（时点报价，非官方价）

| 项目 | 报价 | 时点 | 来源 |
|---|---|---|---|
| H20 96GB 八卡模组（全新，华南现货） | **140 万元/套**（折单卡约 17.5 万元） | 2026-09-03 | [SMM 上海有色网](https://news.smm.cn/news/104094410) |
| H200 PCIe 单卡（华南现货） | 38.5 万元/张 | 2026-09-03 | 同上 |
| H20 二手八卡模组 / 整机 / 全新整机 | 约 125 万 / 190 万 / 240 万元【未核实，来自搜索结果摘要，原页未抓取】 | 2026-09 | SMM 快讯（[104103736](https://news.smm.cn/news/104103736)、[Mysteel 算力](https://suanli.mysteel.com/a/26090811/692F454017594E91.html)） |
| H20 96G 整机租赁 | 约 3.8 万–4.6 万元/台/月（含税含柜电）【未核实，同上】 | 2026-09 | 同上 |
| 昆仑芯 R200 | 2650 元 | 2026 | SMM 算力日报【未核实，原页未抓取】 |
| 昇腾 910B 八卡整机 / 月租 | 约 170 万元起 / 1.2 万–1.7 万元/台/月【未核实，同上】 | — | 同上 |
| 天数智芯 天垓150 单卡 | ￥80,759 | 2025-11 | [AI柠檬汇总](https://blog.ailemon.net/2025/10/31/national-ai-chip-param-info-collection/) |
| H20 单卡（历史）/ 八卡服务器（历史） | $12,000–15,000 / $200,000；同期 910B 约 $17,000 | **2024-02（已严重过时，仅示意「国产卡与 H20 曾同价位」）** | [Tom's Hardware](https://www.tomshardware.com/pc-components/gpus/new-nvidia-ai-gpus-designed-to-get-around-us-export-bans-come-to-china-h20-l20-and-l2-to-fill-void-left-by-restricted-models) |

> X 平台上有从业者称「一台部署本地大模型的国产人工智能一体机都是几十万起步」（[@huanjue2026, 2026-09-12](https://x.com/huanjue2026/status/2098610006690279623)），与上述整机量级一致，但属**个人说法，非厂商报价**【未核实】。

### 2.5 一体机（All-in-One Appliance）

- **形态**：厂商（华为、新华三、浪潮、联想、宝德、神州数码、中科曙光等）把「服务器 + 加速卡 + 预装推理框架 + 模型权重 + 管理/知识库平台」打包交付，开箱即用。2025 年 DeepSeek 爆火后成为政企采购主流 SKU。
- **本次可核验的事实**：
  - 华为 **Atlas 850E 风冷超节点**本身就是「免液冷改造、标准机柜直接上架、单超节点扩展至 96 卡」的一体机式产品 —— [华为新闻](https://www.huawei.com/cn/news/2026/7/atlas-950-superpod)。
  - X 平台从业者反映「前几年大模型本地部署的一体机卖得非常好」（[@wateru2026](https://x.com/wateru2026/status/2098551321775509514)）【个人说法，未核实】。
- **具体型号与报价（如「DeepSeek 一体机 115 万 / 520 万」等）本次未能从一手厂商页面核实 →【未核实】**，不在此罗列。建议直接向厂商索取带配置清单的正式报价单，并在合同中明确：卡型与数量、模型版本与量化格式、并发/TTFT/TPS 的 SLA、国产化率证明、后续模型升级的服务条款。
- **一体机的真实取舍**：优点是交付快、单一责任主体、通过信创清单；缺点是**卡型锁死、扩展性差、模型更新受制于厂商、单位算力溢价通常 20–50%【估算】**。建议：**PoC/部门级用一体机，企业级主平台自建。**

---

## 3. 推理服务与加速

### 3.1 框架选型对照

| 框架 | 定位 | 国产卡支持 | 适用 | 来源 |
|---|---|---|---|---|
| **vLLM**（v0.31.0+） | 事实标准。PagedAttention、连续批处理、prefix caching、chunked prefill、PD 分离、render/derender 前端分离、>12 种 KV connector | 昇腾（vllm-ascend）、海光（`vllm 0.8.5.post1+das...dtk`）、摩尔线程（`...musa`）、昆仑芯（容器内置 vLLM 0.7.x，新镜像已支持 DeepEP low-latency + MTP） | **企业主力**，生态最全 | [vLLM 分离式服务指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)、[海光文档](https://moark.com/docs/compute/clusters_gpu/hygon_gpu)、[摩尔线程文档](https://moark.com/docs/compute/clusters_gpu/moore_gpu)、[腾讯云昆仑芯实践](https://cloud.tencent.com/document/product/1397/127039) |
| **SGLang** | RadixAttention/前缀缓存见长，PD 分离成熟（Mooncake / NIXL / **Ascend memfabric-hybrid** 三种传输后端），router 内置 PD 负载均衡 | **昇腾原生一等公民**（`--attention-backend ascend --device npu`，含 DeepEP、NEXTN 投机解码）、昆仑芯（`--attention-backend kunlun`）、摩尔线程 | 高吞吐、长共享前缀（RAG、Agent）、国产卡 PD 分离 | [SGLang PD 文档](https://docs.sglang.io/docs/advanced_features/pd_disaggregation) |
| **TensorRT-LLM** | NVIDIA 原生极致优化 | 仅 NVIDIA | 纯 N 卡、追求极限延迟、已上 Triton 的团队 |【本次未抓取官方文档 →未核实】 |
| **LMDeploy**（v0.18.0, 2026-09） | 上海 AI Lab，TurboMind 引擎；支持 Llama/InternLM/Qwen/DeepSeek/GLM/gpt-oss 及多模态，compressed-tensors INT4（group 32/128），W4A16 + piecewise CUDA Graph，**Ascend GLM-5.2 适配** | 昇腾 | W4A16 低延迟、国产模型 | 搜索结果（[LMDeploy 支持模型](https://lmdeploy.readthedocs.io/en/latest/supported_models/supported_models.html)、[PR #4429](https://github.com/InternLM/lmdeploy/pull/4429)）【原页未抓取 →未核实】 |
| **llama.cpp / Ollama（GGUF）** | CPU/消费显卡/Apple Silicon，2–8 bit 多档量化 | — | 单机、边缘、离线笔记本、PoC | — |
| **MindIE / vLLM-Ascend** | 华为昇腾官方推理栈，依托 CANN（已开源 67 个项目、1244 万行代码） | 昇腾专用 | 昇腾集群生产 | [华为 WAIC 新闻](https://www.huawei.com/cn/news/2026/7/atlas-950-superpod) |
| **Xinference**（v4.0.0 / v3.5.0） | 多模型统一托管（LLM + Embedding + Rerank + 语音 + 图像），内置模型启动建议与显存元数据 | 依赖底层引擎 | **中小企业「一个平台管所有模型」的最省事方案** | 搜索结果（[Release Notes](https://inference.readthedocs.io/en/stable/getting_started/release_notes.html)）【原页未抓取 →未核实】 |
| **KTransformers** | CPU+GPU 异构卸载跑超大 MoE | — | 显存不足时的「省钱跑大模型」路线 |【未核实】 |

### 3.2 核心加速技术

**（1）连续批处理（Continuous Batching）+ PagedAttention**：vLLM 的基础能力，无需调参即默认开启；量化 + 连续批处理的协同效应见 §1.3（异步场景 W8A8 吞吐最高，8B 模型 INT8 在 A6000 上 QPS 提升 1.38×、70B 提升 1.91×、405B H100 FP8 提升 3.04×）——来源：[arXiv:2411.02355](https://arxiv.org/html/2411.02355v4) 表 6。

**（2）Chunked Prefill**：把长 prompt 切块与解码交错，缓解 prefill 阻塞。vLLM 官方结论：**「块大小取决于流量特征，你会不断重调」**，所以在 SLO 严格场景不如直接做 PD 分离。

**（3）投机解码 / MTP**
- DeepSeek-V4 自带 **DSpark** 投机解码模块；DeepSeek-R1 系列用 **MTP/NextN**（昆仑芯实践中 `--speculative-config '{"method":"deepseek_mtp","num_speculative_tokens":3}'`）。
- SGLang 昇腾实践用 **NEXTN**（`--speculative-num-steps 3 --speculative-eagle-topk 1 --speculative-num-draft-tokens 4`）与 **EAGLE/多层 EAGLE**；昆仑芯 MiniMax-M3 用 **EAGLE3 草稿模型**（`--speculative-num-steps 3`）。
- vLLM Qwen3.8 实测：**MTP 在 KV 空间充足时显著提升解码吞吐；并发升高、KV 吃紧后反而被无 MTP 的大 KV 拓扑反超**。
- 来源：[腾讯云昆仑芯实践](https://cloud.tencent.com/document/product/1397/127039)、[SGLang PD 文档](https://docs.sglang.io/docs/advanced_features/pd_disaggregation)、[vLLM Qwen3.8 PD](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)。
- **工程提醒**：投机解码需要额外显存放草稿模型 + 推测 token 的 KV 槽位；并发高时收益递减，**必须按自己的并发档位实测**。

**（4）并行策略（TP / EP / PP / DP-Attention / CP）**
- vLLM 在 GB300 上对 Qwen3.8-2.4T 的实测结论：
  - **解码**：追极限吞吐用 **TP4DP4+EP**（权重每卡仅 108.71 GiB，留出 133 GiB KV，单引擎并发上限 ~180）；低并发用 **TEP8 + MTP**；并发 512–1024 用 TEP8；更高并发 TP4DP4+EP 反超。
  - **预填充**：低并发 **TP4DP2+EP** 最佳，高并发 **TP2DP4+EP** 占优。
  - 为什么：TP8 时每卡权重 169.28 GiB，KV 只剩 83 GiB，并发上限掉到 ~112。**权重分片方式直接决定并发天花板。**
- MoE all2all 用 **DeepEP**（`--moe-a2a-backend deepep`，预填充 `normal` 模式、解码 `low_latency` 模式）。
- 来源：[vLLM Qwen3.8 PD 博客](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)、[SGLang PD 文档](https://docs.sglang.io/docs/advanced_features/pd_disaggregation)。

**（5）PD 分离（Prefill/Decode Disaggregation）—— 本报告重点**

原理：Prefill 计算密集、决定 TTFT；Decode 访存密集、决定 ITL。同卡混跑时一条 8k 长 prompt 会卡住所有解码流。分离后两侧可**独立选并行度、独立扩缩容**。

| 场景 | 实测收益 | 硬件 | 来源 |
|---|---|---|---|
| Qwen2.5-7B，8k prompt / 256 输出 | 并发 0.4 req/s 时，合置 p99 ITL 从 23ms 飙到 169ms、2 req/s 时 263ms；**PD 分离 p99 始终 25–52ms（约 6× 改善）**，中位 ITL 两者相同（21–24ms） | 2×L40S 48GB PCIe | [vLLM 指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide) |
| Qwen3-235B-A22B-FP8 @ 8 req/s | 满足 TTFT≤1s 且 ITL≤50ms 的请求 **73/100 vs 合置 30/100 → goodput ≈ 2.4×** | 单节点 8×MI300X（MoRI-IO） | 同上 |
| gpt-oss-120b | 端到端平均延迟 **-59%**、P95 **-67%**（对比同样 GPU 做聚合副本） | 16×H200（llm-d） | 同上 |
| Qwen3.8-2.4T，ISL/OSL 8192/1024 | 高吞吐侧 **5,000 total tokens/s/GPU**；低延迟侧 **180 gen tok/s/user**；GSM8K 精度各配置均 95%（分离不掉精度） | GB300 NVL72 | [vLLM Qwen3.8 PD](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving) |
| Qwen3.8 Day-0 | **5,126 tok/s/GPU**（预填充 2×PP6，解码 DP2-attn/TP4/EP8） | — | 搜索结果（[LMSYS 博客](https://www.lmsys.org/blog/2026-08-12-qwen3-8-day0-support/)）【原页未抓取 →未核实】 |

> ⚠️ **PD 分离的反面教训（必须写进方案风险章节）**：同一篇 vLLM 实测中，由于 2×L40S **不支持 P2P**（`nvidia-smi topo -p2p r` 返回 `NS`），每条 8k prompt 的 ~470 MB KV 传输耗时约 **1.3 秒**，导致 PD 分离的中位 TTFT 从合置的 0.7s 恶化到 **2.2s**，在所有速率下都无法满足 2s 的 TTFT 目标。官方给出的判据：
> 1. 先跑 `nvidia-smi topo -p2p r`，P/D 两卡之间必须是 `OK`；
> 2. 看解码侧 `KV Transfer metrics` 的 `Avg xfer time`，**若在数百毫秒量级就先别做 PD**；
> 3. 跨节点必须 RDMA（IB 或 RoCE），同节点必须 NVLink 或 GPU P2P。
> 4. 参考量级：Llama-3.1-70B BF16 每 token KV 320 KiB → 1 万 token prompt 需搬 ~3 GB，在 400 Gb/s 线速下理论就要 **65 ms**，且全部计入 TTFT。
> 来源：[vLLM 分离式服务指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)。

**什么时候该/不该做 PD 分离（vLLM 官方决策表，直译）**

| 你的处境 | 建议 |
|---|---|
| 生产负载下 ITL p99 打不住 SLO | **做分离**（主要用例） |
| 长 prompt + 高并发 | 做分离，前提是 KV 传输够快 |
| 多轮对话 / Agent 上下文不断增长 | 做分离，并开 `bidirectional_kv_xfer`，配合 LMCache 或 Mooncake 共享 KV |
| GPU 节点 CPU 画像里出现模板化/分词/解析 | 拆出 render 层（CPU-only） |
| TTFT 是主约束 | **保持合置**或先量测——传输成本落在每一个首 token 上 |
| KV 传输慢 | 先修，修不好就别分离 |
| 低流量 / 突发 / 对延迟不敏感 | **保持合置** |

**生产化编排**：不要用 vLLM 仓库 `tests/` 里的 toy proxy。生产用 **llm-d**（用 vLLM 原生 remote prefill/decode + NIXL，router 与 EPP 调度器选端点）、**NVIDIA Dynamo**（K8s，自带 router 与 planner）、**KServe `LLMInferenceService`**（基于 llm-d）、**vLLM production-stack**（Helm）或 **AIBrix**（控制面）。来源：同上。

**（6）KV cache 优化**
- **Prefix caching / RadixAttention**：对系统提示词长、RAG 前缀固定的场景收益最大。**注意**：vLLM 官方警告，做 PD 基准测试时若复用 `--seed` 必须加 `--no-enable-prefix-caching`，否则缓存命中会掩盖你想测的 prefill 干扰。
- **KV offload / 共享 KV**：LMCache、Mooncake（Moonshot 开源）、FlexKV、AMD MoRI-IO 均已上游，另有 `MultiConnector` 可串联。
- **双向 KV 传输**（`bidirectional_kv_xfer`）：多轮对话第二轮起，prefill 从 decode 把上一轮生成的 block 拉回来而不是重算。**重大坑**：推理模型（reasoning）的 decode block 里含 thinking trace，若下一轮 chat template 自动丢弃 thinking（Qwen3 模板即如此），prompt 与 block 对不上会产生**错误输出而非仅仅变慢**，vLLM 目前不会检测这种不匹配（issue #43094）。→ **开启前必须检查 chat template**。
- **异构 TP 的 staging buffer**（SGLang）：P/D 两侧 TP 不同时，用 GPU staging buffer 聚合 KV head 分片再批量 RDMA，高并发下吞吐提升 **2–5×**，与同构 TP 基线差距 ~5% 以内；但 **MLA 模型（DeepSeek-V2/V3）不能开**。
- **block size 必须两侧一致**：P/D 各自独立计算 block size，不一致就无法传输，需手工 `--block-size`。
- 来源：[vLLM 指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)、[SGLang PD 文档](https://docs.sglang.io/docs/advanced_features/pd_disaggregation)、[vLLM Qwen3.8 PD](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)。

**（7）前端（render/derender）分离**：把 chat 模板化、分词、反分词、reasoning/tool-call 解析搬到无 GPU 的 CPU 层。实测：9k token 的 Qwen2.5-7B chat prompt 模板化+分词仅约 **15 ms CPU**，单 render 服务默认配置可达 **73 req/s（仅用一核多一点）**。**但流式 + 解析器场景代价很大**：解析器状态不可序列化，每个 chunk 都要重建解析器并重放 token 历史，长生成下是 O(n³) 字符工作量，实测 **约 9× 的进程内 CPU、E2E p50 +15%**，缓存层尚在路上（#57571）；`--renderer-num-workers` 默认只有 1，必须调大。来源：[vLLM 指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)。

### 3.3 压测与容量规划方法（可直接抄）

```bash
# vLLM：按 goodput（同时满足 TTFT/TPOT）而不是峰值吞吐来定容量
vllm bench serve --model <你的模型> --port 8192 \
  --dataset-name random --random-input-len 8192 --random-output-len 256 --ignore-eos \
  --request-rate 0.4 --num-prompts 100 \
  --percentile-metrics ttft,tpot,itl --metric-percentiles 50,99 \
  --goodput ttft:2000 tpot:30 \
  --no-enable-prefix-caching        # 复用 seed 时必加
```
```bash
# SGLang：长共享前缀（RAG）场景的压测
python3 -m sglang.bench_serving --backend sglang --dataset-name generated-shared-prefix \
  --gsp-num-groups 4 --gsp-prompts-per-group 16 \
  --gsp-system-prompt-len 54000 --gsp-question-len 15000 --gsp-output-len 500 \
  --max-concurrency 4 --num-prompts 64 --request-rate 1
```
来源：[vLLM 指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)、[腾讯云昆仑芯实践](https://cloud.tencent.com/document/product/1397/127039)。

**方法论要点**：① 先分别测 prefill-only（ISL/OSL=8192/2）与 decode-only（ISL/OSL=1/1000）选出各自最优拓扑，再做 PD 组合，可大幅减少搜索空间；② 容量按 **goodput** 而非 TPS 定；③ 每个配置先跑 GSM8K 等精度校验再测性能（vLLM 对所有 PD 配置均验到 95%）。来源：[vLLM Qwen3.8 PD](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)。

---

## 4. 安全、合规、高可用与应用集成

### 4.1 中国监管框架：企业私有化部署到底要不要备案？

| 法规/标准 | 关键内容（与私有化部署相关） | 生效 | 来源 |
|---|---|---|---|
| **《生成式人工智能服务管理暂行办法》**（七部门令第 15 号） | **第二条第三款：行业组织、企业、教育和科研机构、公共文化机构、有关专业机构等研发、应用生成式 AI 技术，未向境内公众提供生成式 AI 服务的，不适用本办法。**<br>第六条：鼓励采用**安全可信的芯片、软件、工具、算力和数据资源**（信创导向的法规依据）。<br>第七条：训练数据须合法来源、不侵害知识产权、涉个人信息须取得同意。<br>第十一条：**不得收集非必要个人信息，不得非法留存可识别使用者身份的输入信息和使用记录**（直接约束你的 prompt 日志保留策略）。<br>第十二条：按深度合成规定对生成内容标识。<br>第十七条：具有**舆论属性或社会动员能力**的服务，须开展安全评估并履行算法备案。 | 2023-08-15 | [中央网信办全文](https://www.cac.gov.cn/2023-07/13/c_1690898327029107.htm)、[国务院公报版](https://www.gov.cn/gongbao/2023/issue_10666/202308/content_6900864.html) |
| **GB/T 22239-2019《信息安全技术 网络安全等级保护基本要求》（等保 2.0）** | 现行有效，推荐性国标，是定级备案与测评的技术基线 | 2019-12-01 实施 | [国家标准全文公开系统](https://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=BAFB47E8874764186BDB7865E8344DAF) |
| **《人工智能生成合成内容标识办法》** + 配套**强制性国家标准《网络安全技术 人工智能生成合成内容标识方法》**（市场监管总局/国标委 2025 年第 3 号公告批准发布） | 显式标识（用户可明显感知）+ 隐式标识（文件元数据）双轨；任何组织个人**不得恶意删除、篡改、伪造、隐匿**标识；配套《服务提供者编码规则》实践指南（网安秘字〔2025〕29 号） | 2025-09-01 | [中央网信办发布稿](https://www.cac.gov.cn/2025-03/14/c_1743654685899683.htm) |

**实操结论（建议请法务复核）**
1. **纯内部私有化部署（不向境内公众提供服务）→ 一般不落入第 15 号令的适用范围**，不需要大模型备案/上线安全评估。这是私有化相较公有云 API 的最大合规红利。
2. 但**只要出现以下任一情形就要重新评估**：对外部客户/公众开放（含小程序、公众号、官网客服）、具有舆论属性或社会动员能力、生成内容对外传播。
3. 无论是否备案，**《网络安全法》《数据安全法》《个人信息保护法》与等保 2.0 始终适用**。
4. 对外输出 AI 生成内容时，标识义务是**强制性国标**，不是推荐性要求。
5. 第十一条对「输入信息和使用记录」的留存限制与审计需求存在张力：**审计日志要留，但不能超范围留存可识别身份的输入内容** → 建议做「身份 ID 与内容分库 + 内容脱敏/哈希 + 分级保留期」。

### 4.2 等保 2.0 视角下的 LLM 平台控制项映射【本节为本报告基于 GB/T 22239 框架做的工程映射，非标准原文条款号】

| 等保域 | LLM 平台落地措施 |
|---|---|
| 安全物理环境 | GPU 机房分区、门禁、液冷/风冷改造评估（Atlas 850E 可免改造） |
| 安全通信网络 | 推理集群置于内网安全域；**RDMA/IB 存储与 KV 传输网与业务网物理或 VLAN 隔离**；对外仅暴露网关 |
| 安全区域边界 | 单一入口网关（LiteLLM/Higress/Kong），禁止模型服务直接对外；出向默认拒绝（阻断模型/框架自动拉取外网权重） |
| 安全计算环境 | 容器最小权限（注意：昆仑芯 P800 要求 `--privileged`，需补偿控制）、MIG/多实例隔离、模型权重文件只读挂载 + 校验 |
| 安全管理中心 | 集中日志、集中监控、集中密钥（虚拟 Key）、集中配额 |
| 身份鉴别与访问控制 | SSO/OIDC + LDAP/AD + RBAC + 按团队/用户的模型白名单与配额 |
| 安全审计 | 全量请求审计（谁、何时、用哪个模型、消耗多少 token、命中哪条护栏），审计日志防篡改与独立留存 |
| 数据完整性/保密性 | 传输 TLS、落盘加密、向量库与原文分权限 |
| 备份恢复 | 模型权重与向量库异地备份，推理无状态副本 |

### 4.3 提示注入与输出过滤

**威胁模型（OWASP Top 10 for LLM Applications 2025，LLM01:2025 Prompt Injection）**
- **直接注入**：用户 prompt 直接改变模型行为（含越狱）。
- **间接注入**：模型从**网页、文件、RAG 知识库**读入的外部内容里藏着指令 —— **这正是企业 RAG 的头号风险**。官方场景 #4 即「攻击者修改 RAG 语料库中的文档，用户查询命中后，恶意指令改变模型输出」。
- 多模态注入：指令藏在图片里（场景 #7）；还有载荷拆分（#6）、对抗后缀（#8）、多语言/Base64 混淆（#9）。
- 影响：敏感信息泄露、系统提示词泄露、越权调用模型可用的函数、**在连接系统中执行任意命令**、操纵关键决策。
- **官方明确承认：鉴于生成式 AI 的随机性本质，目前尚不清楚是否存在万无一失的注入防护方法**（原文 "it is unclear if there are fool-proof methods of prevention"）。
- 来源：[OWASP GenAI LLM01:2025](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)（另有 PDF 版 [OWASP Top 10 for LLMs v2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf)）。

**OWASP 给出的七条缓解措施（直译要点）**：① 在系统提示中约束模型角色、能力与边界，并要求忽略改写核心指令的企图；② 定义并用确定性代码校验输出格式；③ 输入输出双向过滤（语义过滤 + 字符串检查 + RAG Triad 评估上下文相关性/有据性/问答相关性）；④ **最小权限**：应用自己持有 API token，功能在代码里实现而不是交给模型；⑤ **高风险操作必须人工审批**；⑥ **隔离并明确标注外部不可信内容**；⑦ 定期对抗测试，把模型当作不可信用户来验证信任边界。

> **本报告的工程转译**：**不要把安全寄托在提示词上。授权必须在模型之外强制执行。** 对企业 RAG，至少做到：检索结果以「数据」而非「指令」的结构化形式注入；对入库文档做注入特征扫描；工具调用走白名单 + 参数校验 + 二次确认；输出侧做敏感信息（身份证、手机号、密钥、内部代号）正则+模型双重脱敏。

### 4.4 模型供应链安全

- **只从可信源取权重**：ModelScope（国内最快）、Hugging Face 官方组织（`deepseek-ai`、`Qwen`、`zai-org`、`moonshotai`、`MiniMaxAI`）。社区有人在 X 上整理了下载指引（[@Poorqym1](https://x.com/Poorqym1/status/2109684993681453099)）【个人整理，未核实】。
- **必做**：校验文件哈希 / safetensors 格式（避免 pickle 反序列化 RCE）；审阅 `trust_remote_code` 所引入的自定义建模代码 —— 注意本报告引用的几乎所有国产大模型启动命令都带 `--trust-remote-code`，**这意味着你在执行模型仓库里的任意 Python 代码**，必须在隔离环境人工审计一次。
- **锁定推理框架版本与镜像**：国产卡定制 wheel 被 `pip install` 覆盖即不可用（见 §2.2）。
- **离线环境**：权重与镜像经内部制品库（Harbor/Nexus）二次分发，推理节点出向默认拒绝。
- **许可证与出口合规**：留存每个模型的 LICENSE 原文与下载时间戳。

### 4.5 网关与应用层

| 组件 | 能力（已核验） | 来源 |
|---|---|---|
| **LiteLLM Proxy** | **虚拟密钥**（`/key/generate`，按 key 限定 models / MCP servers / 管理路由 `allowed_routes`）；**花费跟踪**（按 key / user / team，`/key\|user\|team/info`）；**预算与限速**（`max_budget`、`budget_duration`、`tpm_limit`、`rpm_limit`、`max_parallel_requests`，支持 `upperbound_key_generate_params` 作为硬上限兼默认值）；**模型别名**（灰度升降级不改客户端）；**密钥轮转**（`/key/{k}/regenerate`，支持 `grace_period`；`LITELLM_KEY_ROTATION_ENABLED` + `auto_rotate/rotation_interval` 定时轮转 —— **企业版功能**）；`custom_key_policy` 单钩子统一校验所有 key 操作（可强制「所有 key 必须 7 天内过期」）；`overwrite_user_with_key_hash` 把外发 `user` 字段强制改写为 key 的 sha256 哈希（防客户端伪造身份）；自定义 key 头 `litellm_key_header_name` | [LiteLLM Virtual Keys 文档](https://docs.litellm.ai/docs/proxy/virtual_keys) |
| **Higress AI 网关** | 多模型代理、`ai-token-ratelimit`（按 token 限流）、`ai-security-guard`（内容安全）等 AI 插件 | [Higress AI 场景文档](https://higress.ai/docs/ai/scene-guide/multi-proxy/)（本次抓取过文档站，具体插件参数建议以最新文档为准） |
| **Kong AI Gateway（3.8+）** | 语义缓存（embedding + Redis 8+/Valkey 8.1+ 向量存储）、AI Semantic Prompt Guard（语义允许/拒绝规则，命中返回 403）、AI Rate Limiting Advanced（local/cluster/Redis 策略，支持 token、成本、自定义 prompt 计数） | 搜索结果（[语义缓存](https://developer.konghq.com/plugins/ai-semantic-cache)、[Prompt Guard](https://developer.konghq.com/ai-gateway/policies/ai-semantic-prompt-guard/)、[限流](https://developer.konghq.com/ai-gateway/policies/ai-rate-limiting-advanced/)）【原页未抓取 →未核实】 |
| **Open WebUI** | **自建/私有云/气隙部署**；**无状态应用层 + 共享数据库/存储 + Redis 协调会话与实时事件 → 多副本横向扩展与多节点高可用**；LDAP / Active Directory / SSO 集成；**免费版即可内部无限用户使用，只有白标/改品牌/企业专属功能才需要企业许可**；案例：三星半导体、Astellas 制药（3000+ 员工、420+ 自定义模型）、美因茨大学（3 万学生 + 5 千员工全自研开源栈） | [Open WebUI Enterprise](https://docs.openwebui.com/enterprise/) |
| Dify / RAGFlow / Xinference | 应用编排、RAG 流水线、多模型托管 | 【本次未逐页核实 →未核实】 |

**推荐的分层**：`Open WebUI / 自研业务系统（用户入口）` → `LiteLLM 或 Higress/Kong（统一 OpenAI 协议入口：鉴权、配额、审计、护栏、路由）` → `vLLM / SGLang / MindIE 实例池（PD 分离、多副本）` → `模型权重（只读）+ 向量库`。

### 4.6 高可用与监控

- **无状态层**：Open WebUI、LiteLLM、render 层（vLLM `vllm launch render` 不保存任何逐请求状态，流式亦然）均可放在普通负载均衡器后按 CPU 扩缩容。
- **引擎层**：多副本 + 健康检查。SGLang router 支持 `--health-check-interval-secs`；PD 分离下 decode 侧有心跳机制（`SGLANG_DISAGGREGATION_HEARTBEAT_INTERVAL` 默认 5s、`..._MAX_FAILURE` 默认 2 次失败即标记 prefill 下线）。
- **故障处理**：vLLM `kv_load_failure_policy` 默认 `fail`（请求报错），可改 `recompute`（decode 自己重算 KV，慢但请求能活）；`kv_lease_duration` 默认 30s 控制 prefill 为 decode 保留 block 的时长，**高负载下这是最常需要调的超时**。
- **必须监控的指标**：TTFT、TPOT/ITL（p50/p99）、goodput、队列深度、KV cache 利用率、**`KV Transfer metrics` 的 `Avg xfer time`**、GPU 利用率/显存/温度/功耗、限流与护栏命中率、每团队 token 消耗与预算。
- **硬件侧监控命令**：NVIDIA `nvidia-smi`；海光 `hy-smi`（`--showtopo`、`--showxhclbw`、`--showpagesinfo` 看 ECC 退役页、`-idmon` 滚动监控）；摩尔线程 `mthreads-gmi`（`topo -m`、`mtlink -s`、`-q -d MEMORY,UTILIZATION,POWER,TEMPERATURE --json` 便于接 Prometheus）；昆仑芯 `xpu-smi` / `xpu-smi topo -m`。来源：[海光文档](https://moark.com/docs/compute/clusters_gpu/hygon_gpu)、[摩尔线程文档](https://moark.com/docs/compute/clusters_gpu/moore_gpu)、[腾讯云昆仑芯实践](https://cloud.tencent.com/document/product/1397/127039)。
- **容量与弹性**：按 goodput 规划（§3.3）；预留 10–20% 余量应对 CUDA graph 估算偏差导致的 OOM（vLLM 实测中部分配置 graph 实际用量**超出**预估，正是 `gpu_memory_utilization` 默认 0.92 的那 8% 在兜底）。

### 4.7 RAG 与知识库集成要点

1. **向量库与权限**：检索必须带用户/部门权限过滤（row-level ACL），否则 RAG 会成为**越权读取内部文档的捷径**——这是私有化部署最常见的真实数据泄露路径，比模型本身的风险高得多。
2. **把 prefix caching 用起来**：固定系统提示 + 固定知识前缀的场景，SGLang RadixAttention / vLLM prefix caching 收益最大；压测请用 `generated-shared-prefix` 数据集（§3.3）。
3. **长上下文 ≠ 不要 RAG**：1M 上下文模型把全部文档塞进去，KV cache 成本是线性的（§2.3 的 759 MiB/请求 就是 8K 输入的结果），并发会断崖下跌。**RAG 仍然是省钱的那条路。**
4. **配套模型别忘了**：Embedding / Rerank / OCR / ASR 也要私有化，显存要单独规划（通常 1–2 张 L20/RTX PRO 6000 足够），Xinference 可统一托管。
5. **间接提示注入防护**：见 §4.3。入库即扫描，检索结果结构化注入，工具调用白名单。

---

## 5. 参考架构与选型建议

> 以下三套架构的**硬件配置、容量与价格均为【估算】**，价格依据 §2.4 的时点报价推算，不含机房改造、网络设备、存储、实施与三年运维。**务必以厂商正式报价与自己的 POC 压测结果为准。**

### 5.1 小型（部门级 / PoC，50–200 人，日均 <5 万次请求）

| 项 | 配置 |
|---|---|
| 模型 | **Qwen3.8-27B（Apache-2.0，原生多模态）** 或 Qwen3.6-35B-A3B（Apache-2.0，MoE，解码更快）；FP8 量化；配 bge 系列 Embedding + Rerank |
| 硬件 | 1 台 2×RTX PRO 6000 Blackwell Server Edition（96GB×2）服务器；或 1 台 4×L20（48GB×4）；或 2×昇腾 910B |
| 推理栈 | **vLLM 合置部署**（不做 PD 分离）+ 开启 prefix caching、chunked prefill |
| 应用层 | Open WebUI（免费版，SSO+RBAC+审计）+ LiteLLM（虚拟密钥/预算/审计）+ RAGFlow 或 Dify |
| 网络 | 单内网安全域，网关单点对外，无需 RDMA |
| 高可用 | 单节点（接受计划内停机）或 2 节点主备 |
| 容量【估算】 | 27B FP8 单卡 96GB，可支撑约 30–60 路并发、p50 TTFT <1s（**需实测**） |
| 成本【估算】 | 硬件约 **40 万–80 万元**；软件全开源 0 元；年运维 1 人力 |
| 适用 | 内部知识问答、文档处理、代码助手试点 |

**为什么不做 PD 分离**：vLLM 官方决策表明确——低流量、突发、对延迟不敏感的场景**保持合置**；且 RTX PRO 6000 / L20 **无 NVLink、P2P 受限**，KV 传输会把 TTFT 拖垮（§3.2 的 L40S 反面案例）。

### 5.2 中型（企业级，1,000–5,000 人，日均 50–200 万次请求）

| 项 | 配置 |
|---|---|
| 模型 | 主力 **GLM-5.3-Flash 320B/18B（MIT，原生多模态）** 或 **DeepSeek-V4-Flash 284B/13B（MIT）**；辅以 Qwen3.8-27B 处理简单任务（成本分层路由） |
| 硬件 | 2–4 台 8 卡机：8×H20 96GB（768 GB/台）或 8×昇腾 910C 128GB（1024 GB/台）或 8×沐曦 C600 144GB |
| 互联 | **机内 NVLink/灵衢/MetaXLink；机间 IB 或 RoCE（PD 分离的前置条件）** |
| 推理栈 | vLLM 或 SGLang，**PD 分离**（prefill 池 + decode 池），KV connector 用 NIXL 或 Mooncake，配 LMCache 做跨轮 KV 复用；开 MTP/EAGLE3 投机解码；MoE 走 DeepEP（prefill `normal` / decode `low_latency`） |
| 编排 | Kubernetes + **llm-d** 或 **NVIDIA Dynamo**（不要用 toy proxy），或 SGLang router `--pd-disaggregation` |
| 应用层 | Open WebUI 多副本（无状态 + 共享 PG + Redis）+ LiteLLM 集群 + Higress/Kong 做内容安全与 token 限流 + 自建 RAG 平台 |
| 高可用 | 每层 ≥2 副本；`kv_load_failure_policy: recompute`；心跳与健康检查；模型权重与向量库异地备份 |
| 容量【估算】 | 按 §3.2 实测外推：PD 分离可把 p99 ITL 压到合置的 1/6，goodput 约 2.4×；具体 TPS 必须用 `vllm bench serve --goodput ttft:2000 tpot:30` 实测 |
| 成本【估算】 | 8 卡 H20 整机约 190 万–240 万元/台（§2.4），4 台约 **760 万–960 万元**；昇腾 910B 八卡整机约 170 万元起/台【未核实】；加网络/存储/机房约 +20–30% |
| 适用 | 全员 AI 助手、研发提效、客服知识库、合同/报告处理 |

### 5.3 大型（集团级 / 行业云，万人以上，Agent 密集型）

| 项 | 配置 |
|---|---|
| 模型 | **GLM-5.3 744B/40B** 或 **DeepSeek-V4-Pro-0813 1.6T/49B（MIT，最宽松）** 或 **Kimi K3 2.8T/104B**；多模型并存 + 智能路由（简单任务下沉到 27B/35B） |
| 硬件 | **昇腾 Atlas 950 SuperPoD**（8192 卡、1152 TB 内存、FP8 8 EFLOPS、灵衢 2.0，2026 Q4）或 Atlas 900 A3（384×910C，300 PFLOPS，已商用 750+ 套）；或 16–32 卡级 H20/H200 集群；风冷机房可用 **Atlas 850E（96 卡，10ms 级时延）** |
| 互联 | 超节点统一内存编址（256 TB 寻址、RTT 3 μs）或 IB 400G+ |
| 推理栈 | 多租户多实例；PD 分离 + render/derender 四层分离；跨实例共享 KV（Mooncake/LMCache）；按 SLA 分池（交互池低延迟、批处理池高吞吐） |
| 编排 | K8s + llm-d/Dynamo + KServe `LLMInferenceService`；GPU 池化与配额（MIG 或昇腾虚拟化） |
| 安全 | 多租户隔离、等保三级、全链路审计、独立密钥管理、模型权重完整性校验与签名 |
| 成本【估算】 | 千万元至数亿元量级，**强烈建议分期：先中型架构跑满一年真实负载再扩** |
| 适用 | 集团统一 AI 平台、对内 MaaS、行业大模型 |

### 5.4 选型决策树（简版）

1. **数据能不能出域？** 不能 → 私有化；能且量不大 → 其实公有云 API 更划算（X 上有从业者直言「除了政务军队医疗金融这些行业，现在私有化部署的企业过几年就会发现，私有化部署开源大模型的成本可能够用几十年的」——[@MilesYun66](https://x.com/MilesYun66/status/2104825606928290225)【个人观点】；也有相反观点认为开源模型私有化将成新热点，因为「可用性已经足够好」——[@caozlog](https://x.com/caozlog/status/2104563928101720151)【个人观点】。**两种声音都要听：私有化的理由应当是合规与数据主权，而不是省钱。**）
2. **必须国产化吗？** 是 → 昇腾优先（生态最成熟：CANN 开源、SGLang/vLLM 原生 NPU 后端、超节点商用 750+ 套），次选沐曦 C600（144GB + FP8 + 全国产供应链）/ 摩尔线程 S5000（原生 FP8 + MTLink）；**注意海光 BW1000 不支持 FP8、昆仑芯 P800 官方规格无 FP8**。
3. **许可证能接受吗？** 要绝对宽松 → **DeepSeek 全系（MIT）、GLM-5.3-Flash（MIT）、Qwen3.8-27B/Qwen3.6（Apache-2.0）、Mistral 3（Apache-2.0）、Gemma 4（Apache-2.0）、gpt-oss（Apache-2.0）**；**MiniMax-M3 默认仅非商用，商用需通知/授权，企业务必注意**。
4. **中文/中国业务场景为主吗？** 是 → Qwen / GLM / DeepSeek / Kimi 四选一（§1.2）。
5. **ITL p99 是否打不住 SLO？** 是，且有 NVLink/RDMA → 做 PD 分离；否 → 保持合置并先做 prefix caching + chunked prefill。
6. **模型 ≥7B 吗？** 是 → 放心上 FP8/INT4；否 → 保持 BF16/FP8，别上 INT4（§1.3 的 93.5% 恢复率警示）。

### 5.5 综合对比总表

| 维度 | 小型 | 中型 | 大型 |
|---|---|---|---|
| 人数 | 50–200 | 1,000–5,000 | 10,000+ |
| 模型量级 | 27–35B | 280–320B MoE | 744B–2.8T MoE |
| 卡数【估算】 | 2–4 | 16–32 | 384–8192 |
| 是否 PD 分离 | 否 | **是** | 是（+前端分离） |
| 互联要求 | 无 | 机内 NVLink + 机间 RDMA | 超节点统一内存 / IB 400G+ |
| 编排 | Docker Compose | K8s + llm-d/Dynamo | K8s + 多租户池化 |
| 硬件成本【估算】 | 40 万–80 万元 | 760 万–960 万元 | 千万–亿元级 |
| 等保建议级别 | 二级 | 三级 | 三级及以上 |
| 建设周期【估算】 | 2–6 周 | 3–6 个月 | 6–18 个月 |

---

## 6. 风险清单与常见踩坑

1. **只按权重算显存** → 实测固定开销约 20 GiB/卡 + KV 每请求数百 MiB，低估 30–50%（§2.3）。
2. **没有 RDMA 就上 PD 分离** → TTFT 从 0.7s 恶化到 2.2s（§3.2）。
3. **国产卡上直接加载 HF 的 FP8 权重** → 海光 BW1000 直接报错（§2.2）。
4. **`pip install torch` 覆盖厂商定制 wheel** → 整机不可用（§2.2）。
5. **小模型激进 INT4** → 1.5B 模型恢复率仅 93.5%（§1.3）。
6. **多轮双向 KV 传输 + reasoning 模型模板丢弃 thinking** → 产生**错误输出**而非变慢，框架不报错（§3.2）。
7. **MLA 模型（DeepSeek-V2/V3）开 SGLang staging buffer** → 官方明确不支持（§3.2）。
8. **render 层 `--renderer-num-workers` 保持默认 1 且跑流式解析** → 约 9× CPU、E2E p50 +15%，直接饱和（§3.2）。
9. **用 vLLM 仓库 `tests/` 的 toy proxy 上生产** → 官方明确说那只是开发示例（§3.2）。
10. **把授权逻辑写在系统提示词里** → OWASP 明确「不存在万无一失的注入防护」，授权必须在模型外部强制执行（§4.3）。
11. **RAG 检索不做行级权限过滤** → 越权读取内部文档（§4.7）。
12. **prompt 日志无限期全量留存** → 与第 15 号令第十一条存在合规冲突（§4.1）。
13. **`--trust-remote-code` 不审计** → 等于执行模型仓库里的任意 Python 代码（§4.4）。
14. **一体机锁死卡型** → 模型升级受制于厂商（§2.5）。
15. **按峰值 TPS 而非 goodput 规划容量** → 上线即超 SLO（§3.3）。

---

## 7. 下一步建议（可执行清单）

1. **第 1 周**：明确数据分级与出域边界；法务确认是否属于「向境内公众提供服务」（决定是否备案）；确定目标 SLO（TTFT / ITL p99 / 并发）。
2. **第 2–3 周**：选 2 个候选模型（建议 Qwen3.8-27B + GLM-5.3-Flash 或 DeepSeek-V4-Flash），在租用的同型号硬件上用 `vllm bench serve --goodput` 做 goodput 压测 + 自有业务集精度评测（含量化前后对比）。
3. **第 4 周**：验证国产卡（如需）——精度格式、框架版本、BIOS/容器要求、监控接入。
4. **第 2 个月**：搭网关层（LiteLLM + Open WebUI），跑通审计、配额、SSO、RBAC；建立 prompt 日志的分级保留策略。
5. **第 3 个月**：RAG 落地（含行级权限 + 间接注入防护 + 入库扫描）；建立 Prometheus/Grafana 看板（含 `KV Transfer metrics`）。
6. **第 4–6 个月**：按实测负载决定是否扩到 PD 分离与多副本；准备等保测评材料。

---

## 附录 A：本报告引用的全部一手来源（本次实际抓取）

**模型**：[DeepSeek V4 预览](https://www.deepseek.com/en/news/v4-preview/)｜[DeepSeek-V4-Flash](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)｜[DeepSeek-V4-Pro](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/raw/main/README.md)｜[DeepSeek-V4-Pro-0813](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813/raw/main/README.md)｜[DeepSeek-V3.2](https://huggingface.co/deepseek-ai/DeepSeek-V3.2)｜[Qwen3.8-2.4T-A95B README](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md) / [LICENSE](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/LICENSE)｜[Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B/raw/main/README.md)｜[Qwen3.6-35B-A3B](https://huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/README.md)｜[Qwen3.5-397B-A17B](https://huggingface.co/Qwen/Qwen3.5-397B-A17B/raw/main/README.md)｜[GLM-5 博客](https://z.ai/blog/glm-5?id=GLM5)｜[GLM-5.3 README](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md) / [LICENSE](https://huggingface.co/zai-org/GLM-5.3/raw/main/LICENSE)｜[GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash/raw/main/README.md)｜[GLM 技术报告 arXiv 2602.15763](https://arxiv.org/html/2602.15763v2)｜[Kimi K3 README](https://huggingface.co/moonshotai/Kimi-K3/raw/main/README.md) / [LICENSE](https://huggingface.co/moonshotai/Kimi-K3/raw/main/LICENSE)｜[Kimi K2.5](https://huggingface.co/moonshotai/Kimi-K2.5/raw/main/README.md)｜[MiniMax-M3 README](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/README.md) / [LICENSE](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/LICENSE)｜[MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)｜[Mistral 3](https://mistral.ai/news/mistral-3/)｜[Mistral Large 4](https://mistral.ai/news/mistral-large-4/)｜[Llama 4 Model Card](https://raw.githubusercontent.com/meta-llama/llama-models/main/models/llama4/MODEL_CARD.md)｜[Gemma 4 模型卡](https://ai.google.dev/gemma/docs/core/model_card_4)｜[Gemma 版本列表](https://ai.google.dev/gemma/docs/releases)｜[gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b)

**硬件**：[NVIDIA H200](https://www.nvidia.com/en-us/data-center/h200/)｜[NVIDIA HGX](https://www.nvidia.com/en-us/data-center/hgx/)｜[RTX PRO 6000 Server Edition](https://www.nvidia.com/en-us/data-center/rtx-pro-6000-blackwell-server-edition.md)｜[RTX 5090](https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/rtx-5090/)｜[Tom's：H20/L20/L2 规格与价格](https://www.tomshardware.com/pc-components/gpus/new-nvidia-ai-gpus-designed-to-get-around-us-export-bans-come-to-china-h20-l20-and-l2-to-fill-void-left-by-restricted-models)｜[Tom's：中国批准首批 H200](https://www.tomshardware.com/pc-components/gpus/china-approves-first-nvidia-h200-deliveries-to-bytedance-and-tencent-under-case-by-case-import-licenses)｜[华为 Atlas 950 超节点](https://www.huawei.com/cn/news/2026/7/atlas-950-superpod)｜[徐直军 HC2025 演讲（昇腾路标）](https://www.huawei.com/cn/news/2025/9/hc-xu-keynote-speech)｜[AI柠檬：昇腾参数汇总](https://blog.ailemon.net/2025/05/24/huawei-ascend-npu-params-for-ai/)｜[AI柠檬：国产AI计算卡汇总](https://blog.ailemon.net/2025/10/31/national-ai-chip-param-info-collection/)｜[模力方舟：海光 BW1000](https://moark.com/docs/compute/clusters_gpu/hygon_gpu)｜[模力方舟：摩尔线程 S5000](https://moark.com/docs/compute/clusters_gpu/moore_gpu)｜[腾讯云：昆仑芯 P800 部署实践](https://cloud.tencent.com/document/product/1397/127039)｜[IT之家：沐曦曦云 C600](https://www.ithome.com/0/890/942.htm)｜[SMM：H20 八卡模组报价](https://news.smm.cn/news/104094410)

**推理栈**：[vLLM：分离式服务实践指南](https://vllm.ai/blog/2026-09-29-disaggregated-serving-guide)｜[vLLM：Qwen3.8-2.4T PD 服务](https://vllm.ai/blog/2026-09-21-qwen38-pd-serving)｜[vLLM v0.31.0 发布说明](https://newreleases.io/project/github/vllm-project/vllm/release/v0.31.0)｜[SGLang PD 分离文档](https://docs.sglang.io/docs/advanced_features/pd_disaggregation)

**量化与精度**：[arXiv:2411.02355v4 "Give Me BF16 or Give Me Death?"](https://arxiv.org/html/2411.02355v4)｜[NVIDIA NVFP4 博客](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

**合规与安全**：[《生成式人工智能服务管理暂行办法》(CAC)](https://www.cac.gov.cn/2023-07/13/c_1690898327029107.htm) / [国务院公报版](https://www.gov.cn/gongbao/2023/issue_10666/202308/content_6900864.html)｜[GB/T 22239-2019 等保 2.0](https://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=BAFB47E8874764186BDB7865E8344DAF)｜[《AI 生成合成内容标识办法》及配套强制国标 (CAC)](https://www.cac.gov.cn/2025-03/14/c_1743654685899683.htm)｜[OWASP LLM01:2025 提示注入](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)｜[OWASP Top 10 for LLMs v2025 PDF](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf)

**网关与应用**：[LiteLLM Virtual Keys](https://docs.litellm.ai/docs/proxy/virtual_keys)｜[LiteLLM Enterprise](https://docs.litellm.ai/docs/enterprise)｜[Higress AI 多模型代理](https://higress.ai/docs/ai/scene-guide/multi-proxy/)｜[Open WebUI Enterprise](https://docs.openwebui.com/enterprise/)

**X（社区观感，均为个人观点，非权威数据）**：[@caozlog 论开源私有化成新趋势](https://x.com/caozlog/status/2104563928101720151)｜[@MilesYun66 反方观点](https://x.com/MilesYun66/status/2104825606928290225)｜[@huanjue2026 论国产一体机价格](https://x.com/huanjue2026/status/2098610006690279623)｜[@wateru2026 论一体机销量](https://x.com/wateru2026/status/2098551321775509514)｜[@xiongchun007 论大厂限制闭源模型](https://x.com/xiongchun007/status/2101539785798217764)｜[@tim1998robot 在 910C 上部署 DeepSeek-V4-Flash](https://x.com/tim1998robot/status/2081337929301889345)｜[@Poorqym1 国产开源模型下载指引](https://x.com/Poorqym1/status/2109684993681453099)

---

## 附录 B：未核实事项汇总（决策前必须自行核验）

1. 昇腾 910B 显存带宽 392 GB/s（与同表 910A 的 1.5 TB/s 矛盾，疑为笔误）。
2. 沐曦 C600 的显存带宽与算力具体数值（仅确认 144GB HBM3e + 支持 FP8）。
3. 寒武纪 MLU690 / MLU590 是否支持 FP8（官方未公开披露）。
4. 昆仑芯 P800 显存带宽（官方规格表未列）。
5. H20 二手/整机价格、租赁价格、昇腾 910B 整机价格（来自搜索摘要，原页未抓取）。
6. 「DeepSeek 一体机」等具体型号与报价（未找到一手厂商页面）。
7. TensorRT-LLM、LMDeploy v0.18.0、Xinference v4.0.0 的具体版本能力（来自搜索摘要）。
8. Kong AI Gateway 插件的具体参数与版本要求（来自搜索摘要）。
9. Dify / RAGFlow 的企业级能力细节。
10. NVIDIA Model Optimizer QAD 的 IFBench/MMMU-Pro 具体数据（来自搜索摘要）。
11. MiniMax-M2.5 的许可证归属（HF 页显示 MIT，但 M3 为 COMMUNITY LICENSE，需以仓库 LICENSE 文件为准）。
12. Qwen3.8-Flash-Next 的许可证与 125B/51B 参数构成细节。
13. 昇腾 Atlas 950PR（2026 Q1）与 950DT（2026 Q4）的实际上市与供货情况。
14. 中国境内 H200 进口许可的当前范围（已知首批为字节、腾讯）。
15. 本报告所有标注【估算】的显存、容量、成本与周期数字。

---

*报告结束。编制时间：2026-10-14（UTC+8）。本报告不构成采购建议或法律意见；合规判断请以法务/监管部门正式意见为准。*
