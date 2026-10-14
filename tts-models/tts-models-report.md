# 近一年主流 TTS（文本转语音）模型调研报告（开源 + 商业）

> 撰写日期：2026-10-14（北京时间）｜覆盖窗口：约 2025-10 ~ 2026-10。窗口外但仍作为基线或对照收录的模型（正文中均已标注“窗口外”）：Kokoro-82M（2025-01）、Chatterbox 原版（2025-05）、VibeVoice-1.5B（2025-08）、IndexTTS-2（2025-09-08）、VoxCPM-0.5B（2025-09）、Seed-TTS（2024，仅作评测参照）。
> 写作原则：**所有数字、日期、价格都来自本次实际抓取的页面**（链接见每节及文末“来源清单”）。凡是厂商自报、第三方转述或本次没能核实的内容，都会单独标出来。榜单 Elo 会随投票实时变化，本文用的是 2026-10-14 抓取时的快照。

---

## 0. 一页结论（TL;DR）

1. **人类盲听（Artificial Analysis Provider Voice 榜）商业第一梯队**：Eleven v4 Turbo（1330）≈ Eleven v4（1328）> Qwen-Audio-3.1-TTS-Plus（1300）> Cartesia Sonic 3.6（1286）> Gemini 3.8 Flash TTS（1274）。这几款全是 2026 年 8~9 月发布的。
2. **性价比突出的是阿里 Qwen-Audio-3.1-TTS-Plus**：AA 标价 $19.3/百万字符，在 Provider Voice 榜排第 3，在 Controlled Voice（克隆音色）榜排第 2；Eleven v4 标价 $80/百万字符。如果只看单价，Simba 3.2（$6.6，Provider 第 9）和 Inworld Realtime TTS-2 Flash（$10.4，Provider 第 13）更便宜。
3. **开放权重榜首换成了 Fish Audio Drama 3 Preview**（AA 开放权重榜 1243，2026-10），第二是 **Breeze TTS 2**（1221）。两者都**不能无条件商用**：Drama 3 只对年营收低于 50 万美元的主体免费商用，Breeze TTS 2 仅限研究和非商用。
4. **可放心商用的开源中文主力**：**Qwen3-TTS**（Apache-2.0，10 种语言，3 秒克隆，支持指令控制；官方称流式首包最低 97ms）、**VoxCPM2**（Apache-2.0，30 种语言 + 9 种中文方言，48kHz 输出）、**Fun-CosyVoice3-0.5B-2512**（Apache-2.0，9 种语言 + 18 种以上方言，双向流式约 150ms）。
5. **客观指标（Seed-TTS-eval）**：§4 表中各开源模型的 test-zh CER 在 0.54~1.22% 之间；test-en WER 在近一年发布的模型里多为 0.99~1.84%，例外是 Fun-CosyVoice3-0.5B-2512 基础版 2.24（RL 版 1.68）和 VibeVoice-Realtime 2.05；窗口外的 IndexTTS2 为 2.23、VibeVoice-1.5B 为 3.04。闭源参考 Seed-TTS 为 zh 1.12 / en 2.25。也就是说，开源模型的中文 CER 已普遍不高于 Seed-TTS，英文 WER 大多更低。不过这些数字是各家自报或互相转引的，口径不统一，同一模型在不同 README 里的数字也可能不同（见 §4）。
6. **端侧 / CPU**：Kyutai **Pocket TTS**（100M 参数，2 核 CPU，首包约 200ms）、**Kokoro-82M**（Apache-2.0，HF Open TTS 英文 WER 第 1）、**Supertonic 3**（31 种语言，不含中文，OpenRAIL-M）、**Chatterbox-Nano**（MIT）都偏英文和欧洲语言。Kokoro 虽然带 8 个普通话音色，但不支持克隆。**中文端侧目前还缺少经过公开验证的成熟轻量模型**。
7. **注意许可证**：OmniVoice（权重 CC-BY-NC，HF Open TTS 9 语言榜第 1）、Fish S2 Pro（研究许可；Seed-TTS-eval zh 0.54 / en 0.99，转引）、Higgs TTS 3（非商用 + 创作者授权；HF Open TTS 9 语言克隆第 2）在客观指标上表现突出；Breeze TTS 2（非商用）和 Voxtral TTS（CC-BY-NC-4.0）主要在人类盲听 Elo 上有优势（AA 开放权重榜分别为 1221 和 1088），客观榜上并不突出（Breeze 在 HF Open TTS 英文 WER 为 1.72，排第 9；Voxtral 本次没有找到 WER/CER）。**这些模型都不能直接放进商业产品**。

---

## 1. 方法与口径说明

| 数据源 | 类型 | 本次如何使用 | 注意事项 |
|---|---|---|---|
| [Artificial Analysis（AA）Provider Voice 榜](https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice) | 人类盲听 Elo（各家自带音色） | 2026-10-14 抓取全表 | 以 Zonos-v0.1=1000 为锚点；Elo 实时变动，X 上不同日期的帖子引用的数值不同（如 Eleven v4 1319/1321/1328） |
| [AA 开放权重榜](https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice/open-weights) | 同上，只看开放权重模型 | 已抓取 | 开放权重 ≠ 可商用 |
| [AA Controlled Voice 榜](https://artificialanalysis.ai/text-to-speech/leaderboard/controlled-voice) | 所有模型用同样 8 个克隆音色（4 美音 + 4 英音） | 已抓取 | 衡量的是克隆音色下的自然度；锚点为 OpenAudio S1=1000；只有英语口音 |
| [TTS Arena V2](https://tts-agi-tts-arena-v2.hf.space/api/leaderboard) | 社区盲听 Elo | 通过其公开 `/api/leaderboard` JSON 抓取 | **还没收录** Eleven v4、Sonic 3.6、Qwen-Audio-3.x 等最新旗舰，参考价值有限 |
| [HF Open TTS Leaderboard](https://huggingface.co/blog/open-tts-leaderboard)（2026-09-30 发布） | 客观指标：WER/CER（Qwen3-ASR 转写）、RTFx、TTFA、SIM | 博客已抓取；具体数字来自第三方媒体 [creativeainews](https://www.creativeainews.com/articles/open-tts-leaderboard-license-cpu-test-2026/) 的转述（结果版本 30-09-2026） | 只衡量可懂度、速度和相似度，**不衡量自然度** |
| [Seed-TTS-eval](https://github.com/BytedanceSpeech/seed-tts-eval/) | 客观：test-en WER（Whisper-large-v3）、test-zh CER（Paraformer-zh）、SIM（WavLM-large） | 引用各模型 README 自报或转引的数字 | 各家复现条件不同，**只能横向粗比**；SIM 有的写成百分数（75.3），有的写成小数（0.749） |
| 厂商博客 / 文档 / 定价页 | 发布日期、功能、延迟、价格 | 逐一抓取 | 延迟数字几乎都是**厂商自报**，测试硬件和测量口径各不相同 |
| X（Twitter） | 发布动态 | 用 x 命名空间检索 | 只用来交叉印证 AA 官方账号的榜单播报，不作为数字主来源 |

---

## 2. 榜单速览

### 2.1 AA Provider Voice 榜 Top 20（2026-10-14 快照）

| # | 模型 | Elo（±95%CI） | 发布 | AA 标注 API 价（/百万字符） |
|---|---|---|---|---|
| 1 | Eleven v4 Turbo | 1330±18 | 2026-09 | $40.0 |
| 2 | Eleven v4 | 1328±17 | 2026-09 | $80.0 |
| 3 | Qwen-Audio-3.1-TTS-Plus | 1300±17 | 2026-09 | $19.3 |
| 4 | Sonic 3.6（Cartesia） | 1286±15 | 2026-08 | $49.0 |
| 5 | Gemini 3.8 Flash TTS | 1274±15 | 2026-09 | $16.5 |
| 6 | Qwen-Audio-3.0-TTS-Plus | 1268±15 | 2026-09 | $19.3 |
| 7 | Realtime TTS-2（Inworld） | 1260±15 | 2026-08 | $20.8 |
| 8 | Gemini 3.8 Flash-Lite TTS | 1248±14 | 2026-09 | $11.0 |
| 9 | Simba 3.2（Speechify） | 1245±13 | 2026-07 | $6.6 |
| 10 | Fish Audio Drama 3 Preview（开放权重） | 1243±16 | 2026-10 | $15.0 |
| 11 | Breeze TTS 2（开放权重） | 1221±14 | 2026-08 | $34.0 |
| 12 | Luna TTS | 1218±12 | 2026-06 | $15.0 |
| 13 | Realtime TTS-2 Flash（Inworld） | 1218±14 | 2026-08 | $10.4 |
| 14 | Gemini 3.1 Flash TTS | 1213±11 | 2026-04 | $18.3 |
| 15 | StepAudio 2.5 TTS（Aug 2026） | 1211±15 | 2026-08 | $85.0 |
| 16 | v3 Conversational（ElevenLabs） | 1209±14 | 2026-08 | $50.0 |
| 17 | Sonic 3.5 | 1193±12 | 2026-05 | $49.0 |
| 18 | Fish Audio S2.1 Pro（Oct 2026） | 1192±15 | 2026-10 | $15.0 |
| 19 | Soniox TTS Real-Time v2 | 1180±11 | 2026-08 | $14.2 |
| 20 | Eleven v3 | 1176±11 | 2026-02 | $100.0 |

同一张表里的其他相关条目：MiniMax Speech 2.8 HD 1175（$100）、Speech 2.8 Turbo 1152（$60）、Fish Audio S2 Pro 1118、Step Audio EditX 1097、Voxtral TTS 1088、Magpie-Multilingual 357M 1065、Kokoro 82M 1063（表格四舍五入显示为 $0.7，AA FAQ 给出精确值 $0.65/百万字符，全榜最便宜）、Higgs Audio V3 TTS 1039、Chatterbox 1026、VibeVoice 1.5B 957。AA 上名为 “Qwen3 TTS” 的条目（2026-01，933，$13.8）在 Provider 总榜排第 87，“Qwen3 TTS Flash”（2025-09，943，$11.0）排第 84。两者**都不在** AA 开放权重榜的 17 个模型里（见 §2.2），说明 AA 没有把它们归为开放权重条目。它们分别对应哪个具体 API 版本，本文**未核实**。
来源：[AA Provider Voice](https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice)

### 2.2 AA 开放权重榜（2026-10-14，全部 17 个模型）

1 Fish Audio Drama 3 Preview 1243 → 2 Breeze TTS 2 1221 → 3 Fish Audio S2 Pro 1118 → 4 Step Audio EditX（Mar 2026）1097 → 5 Voxtral TTS 1088 → 6 Magpie-Multilingual 357M 1065 → 7 Kokoro 82M v1.0 1063 → 8 Maya1 1047 → 9 OpenAudio S1 Mini 1045 → 10 Higgs Audio V3 TTS 1039 → 11 Chatterbox 1026 → 12 Zonos-v0.1 1000（锚点）→ 13 VibeVoice 1.5B 957 → 14 OpenVoice v2 956 → 15 XTTS v2 916 → 16 StyleTTS 2 895 → 17 MetaVoice v1 848。

AA 的说明是“97 个模型中 17 个为开放权重”。上面已经列出全部 17 个，**其中没有 Qwen3-TTS、Fun-CosyVoice3、VoxCPM2、IndexTTS 这几个开源模型**，所以它们没有 AA 的人类盲听分。总榜上的 “Qwen3 TTS”（933）和 “Qwen3 TTS Flash”（943）也不在这 17 个之中，见 §2.1。来源：[AA 开放权重榜](https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice/open-weights)（2026-10-14 重新抓取核对）

### 2.3 AA Controlled Voice（克隆音色）榜 Top 18

1 HeyGen Voice 1195（2026-10，$30）→ 2 Qwen-Audio-3.1-TTS-Plus 1180 → 3 Eleven v4 Turbo 1163 → 4 Eleven v4 1158 → 5 Sonic 3.6 1143 → 6 Realtime TTS-2 1140 → 7 Qwen-Audio-3.0-TTS-Plus 1125 → 8 Sonic 3.5 1116 → 9 Simba 3.2 1088（$6.6）→ 10 StepAudio 2.5 TTS 1074（AA 标注发布于 Apr 2026）→ 11 Eleven v3 1072 → 12 Realtime TTS-2 Flash 1072（$10.4）→ **13 Gemini 3.8 Flash-Lite TTS 1058±14（$11.0）→ 14 Fish Audio Drama 3 Preview 1058±15** → 15 v3 Conversational 1057 → 16 SpaceXAI TTS 1055 → 17 Gradium TTS（Sep 2026）1054 → **18 Gemini 3.8 Flash TTS 1048±14（$16.5）**。
其他：Speech 2.8 HD 1019（#20）、Breeze TTS 2 1017（#23）、Voxtral TTS 1002（#26）、Fish S2 Pro 1001（#27）、Higgs Audio V3 960（#34）、Chatterbox 929（#38），共 43 个模型。
来源：[AA Controlled Voice](https://artificialanalysis.ai/text-to-speech/leaderboard/controlled-voice)（2026-10-14 重新抓取核对）；AA 官方 X 账号播报 HeyGen Voice 登顶：[x.com/ArtificialAnlys/status/2108608255387935199](https://x.com/ArtificialAnlys/status/2108608255387935199)

> 解读：Gemini 3.8 Flash TTS 在 Provider Voice（自带音色）榜排第 5，在克隆音色榜只排第 18；Qwen-Audio-3.1-TTS-Plus 在两张榜上都在前 3，说明**它的克隆效果和自带音色都很强**。

### 2.4 TTS Arena V2（社区盲听，抓取于 2026-10-14）

公开 API 返回的 `rank` 字段与 Elo 数值大小的顺序**并不一致**，API 也没有说明排序依据（未核实），所以下面两种顺序都列出：

| API rank | 模型 | Elo ± 不确定度 | tier | 票数 |
|---|---|---|---|---|
| 1 | CastleFlow v1.0 | 1561 ± 21 | S | 1643 |
| 2 | Inworld TTS MAX | 1558 ± 21 | S | 1477 |
| 3 | Aurora（匿名模型） | 1566 ± 31 | A | 571 |
| 4 | Papla P1 | 1550 ± 25 | A | 958 |
| 5 | Inworld TTS | 1542 ± 19 | B | 1963 |
| 6 | star-june-2026 | 1540 ± 33 | B | 494 |
| 7 | Hume Octave | 1526 ± 21 | B | 1141 |
| 8 | MiniMax Speech 2.8 HD | 1540 ± 33 | （无） | 502 |
| 0（未正式排名） | Vocu V3.0 | 1564 ± 22 | （无） | 1444 |

> 复核（2026-10-14 重新请求 API）：star-june-2026 和 MiniMax Speech 2.8 HD 的 API 返回值确实都是 elo 1540、uncertainty 33。两者是不同条目：票数分别为 494 和 502，胜率分别为 53.24% 和 53.19%，tier 分别为 B 和无。这不是转录错误。

- **按 Elo 数值排序**：Aurora 1566 > Vocu V3.0 1564 > CastleFlow 1561 > Inworld TTS MAX 1558 > Papla P1 1550 > Inworld TTS 1542 > MiniMax Speech 2.8 HD 1540 = star-june-2026 1540 > Hume Octave 1526。
- 开源模型中，Chatterbox 为 1481（rank 28），Kokoro v1.0 为 1478（rank 29）。
- **这个榜单还没有收录 Eleven v4、Sonic 3.6、Qwen-Audio-3.x、Gemini 3.8 等新模型**，所以本文主要参考 AA 的榜单。来源：[TTS Arena V2 API](https://tts-agi-tts-arena-v2.hf.space/api/leaderboard)

### 2.5 HF Open TTS Leaderboard（客观指标，结果版本 2026-09-30，第三方转述）

- **英文默认音色 WER**：Kokoro-82M 1.46 → Supertonic 3 1.54 → Fish S2 Pro 1.58 → OmniVoice 1.61 → pocket-tts 1.62 → Fun-CosyVoice3 0.5B 1.68 → Chatterbox 1.70 → Qwen3-TTS 1.7B CustomVoice 1.71 → Breeze-TTS-2 1.72 → VibeVoice-Realtime 0.5B 1.82。
- **9 种语言默认音色平均 WER**：OmniVoice 3.25 → Fish S2 Pro 3.68 → Qwen3-TTS 1.7B CustomVoice 4.11。
- **9 种语言克隆（WER / SIM）**：OmniVoice 3.52/74.77 → Higgs TTS 3 3.74/69.08 → Qwen3-TTS 1.7B Base 3.92/72.92；在该视图里 VoxCPM2 的 SIM 最高（75.19）。
- **流式**：Qwen3-TTS 1.7B 用 Photon 引擎在 H200 上 TTFA 为 18.3ms，用 faster-qwen3-tts 为 309.9ms，说明**推理引擎对延迟的影响非常大**；CPU 上 pocket-tts TTFA 272.7ms，Kokoro 913.4ms。
- 来源：HF 官方博客 [Open TTS Leaderboard](https://huggingface.co/blog/open-tts-leaderboard)（方法、Top 模型名）；具体数值转引自 [creativeainews](https://www.creativeainews.com/articles/open-tts-leaderboard-license-cpu-test-2026/)（第三方，作者称数据通过 Space 的 API 拉取）。

---

## 3. 逐模型详解

### 3.A 商业 / 闭源 API

#### 3.A.1 ElevenLabs Eleven v4 / Eleven v4 Turbo
- **发布方 / 时间**：ElevenLabs，2026-09-28（[官方博客](https://elevenlabs.io/blog/eleven-v4)、[TechCrunch](https://techcrunch.com/2026/09/28/elevenlabs-new-v4-speech-model-supports-more-expression-control-and-90-languages/)）。
- **类型**：商业闭源 API。
- **语言**：90+ 种。TechCrunch 报道称，官方表示质量提升最大的是日语、巴西葡语、普通话和粤语。中文的实际质量没有第三方中文盲测，**未核实**。
- **克隆**：Instant Voice Clone 只需约 10 秒音频；v3 不支持的 Professional Voice Clone 在 v4 中恢复。克隆需要音色所有者验证同意（[v4 页面](https://elevenlabs.io/v4)）。
- **情感控制**：内联 Audio Tags（如 `[whispers]`、`[laughs]`、`[door slams]`），可以叠加多个标签；v4 中禁用了 SSML。
- **长篇 / 多说话人**（[v4 页面](https://elevenlabs.io/v4)）：官方称内置多说话人和音效（“multiple speakers and sound effects built-in”）。“Context stitching” 用于在任意长度的脚本中保持节奏和演绎一致，官方说法是“整本有声书听起来像一次录完”。重复生成同一句时，说话人音色保持稳定。
- **延迟 / 流式**：v4 Turbo 中位推理延迟约 100ms，首次出声约 150ms，支持双向流式（官方自报）。单次生成上限 10,000 字符。
- **价格**：AA 标注 v4 Turbo $40/百万字符、v4 $80/百万字符；免费档每月 10,000 credits，付费档 $6/月起（[v4 页面](https://elevenlabs.io/v4)）。
- **榜单**：AA Provider Voice 第 1 / 第 2（1330 / 1328）；Controlled Voice 第 3 / 第 4（1163 / 1158）。

#### 3.A.2 阿里 Qwen-Audio-3.1-TTS（Plus / Flash / Next）与 Qwen-Audio-3.0-TTS-Plus
- **发布方 / 时间**：阿里千问 / 阿里云百炼。Qwen-Audio-3.1 系列发布于 2026-09-23（[IT之家](https://www.ithome.com/1/006/280.htm)）。
- **类型**：商业 API（百炼）。注意它和开源的 Qwen3-TTS 不是同一个东西。
- **语言**：多语种和中文方言；IT之家称支持“同一音色跨语言合成时的自然迁移”。3.1-Flash 文档写明“支持多种语言和方言”。具体语种和方言列表本次**未核实**。
- **克隆 / 情感**：3.0-Plus 文档写明支持 free-style 指令和细粒度标签，可控制情绪、语气、角色、语速、音量；3.1-Flash 文档写明克隆时对带噪声、混响的参考音频更鲁棒。
- **延迟 / 流式**：3.1-Flash 面向实时交互，支持流式（WebSocket）。官方文档**没有给出毫秒数**，未核实。
- **价格（官方文档原价）**：
  - qwen-audio-3.0-tts-plus：北京 **1.4 元/万字符**（约 140 元/百万字符），新加坡 1.49884 元/万字符（[文档](https://help.aliyun.com/zh/model-studio/qwen-audio-3-0-tts-plus)）。
  - qwen-audio-3.1-tts-flash：输入 1.5 元/百万 Token，输出 12 元/百万 Token（[文档](https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-flash)）。
  - qwen-audio-3.1-tts-plus：阿里云帮助中心对应 URL 返回 404，**官方人民币价格未核实**；AA 标注 $19.3/百万字符。
  - qwen-audio-3.1-tts-next（[文档](https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-next)）：统一生成人声、音效、环境音的 Audiogen 模型。文档列出的语种为**中文、英文**，输出为**非流式**；最大输入 3000 字符，单次生成上限为播客 240 秒、其他场景 120 秒；最多 3 条参考音频，每条最长 30 秒。北京价格为输入 6 元/百万 Token、输出 12 元/百万 Token。48kHz 输出这一点出自 IT之家的报道。
- **关于降价**：IT之家称 3.1 发布时 Qwen-Audio 全线降价，其中“TTS 降价约 70%”。但 AA 对 3.0-Plus 和 3.1-Plus 标的都是 $19.3/百万字符（AA 未标注价格的采集时间）。降价前的价格、降价具体针对哪些型号，以及 3.0-Plus 文档里的 1.4 元/万字符是不是降价后的价格，本次都**未核实**，两者不宜直接对比。
- **榜单**：AA Provider Voice 第 3（1300）；Controlled Voice 第 2（1180）。

#### 3.A.3 Cartesia Sonic 3.6
- **发布方 / 时间**：Cartesia，2026-08-27 GA（[官方博客](https://www.cartesia.ai/blog/sonic-3.6)）。
- **语言**：44 种语言、61 个 locale（其中 11 个印度语种 locale），500+ 预置音色，支持 Hinglish 混说。中文质量**没有公开数据**。
- **克隆**：Instant Voice Clone，官方称对口音的保持能力有提升。
- **情感**：官方称美式英语的优势来自“更好的语调、更高的音质和 context-driven emotionality（随上下文调整情感表达）”，并会根据对话上下文调整演绎（厂商自述）。
- **延迟**：官方称 “Replies under 90ms”，生成速度 132 字符/秒（v3 Conversational 为 68）。可以通过 Baseten、SageMaker、Together AI 部署在客户自己的云里。
- **价格**：AA 标注 $49/百万字符。
- **榜单**：AA Provider Voice 第 4（1286），Controlled 第 5（1143）。官方自报美式英语盲测中 92% 优于 Eleven v3（厂商自测）。

#### 3.A.4 Google Gemini 3.8 Flash TTS / Flash-Lite TTS
- **发布方 / 时间**：Google，2026-09-23（[官方博客](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-8-text-to-speech/)）。
- **语言**：100+ 种语言和方言；2,000+ 预置音色。
- **克隆**：Voice replication 需要约 30 秒样本，并且必须提供音色本人的口头同意录音；输出带 SynthID 水印。AI Studio 的复刻功能在伊利诺伊、德州、EEA、英国、瑞士、印度不可用。
- **情感 / 导演控制**：自然语言 Voice Design、逐行舞台指示、原生双说话人场景、backchannel 和非语言声音。官方称在 Hume AI Voice Design Benchmark 上排第 1（71.4，厂商引用）。
- **长篇**：博客称可以“在数小时连续音频中保持高音质、自然节奏和角色音色，speaker drift 极小”，适合播客和有声书（厂商自述）。
- **延迟**：博客**没有给出具体数字**。
- **价格**（[官方定价页](https://ai.google.dev/gemini-api/docs/pricing)）：3.8 Flash TTS 付费档在 2026-12-31 前为文本输入 $0.50/百万 tokens、音频输出 $9.00/百万 tokens（约合 $0.00225 / 10 秒音频）；**2027-01-01 起翻倍**（$1.00 / $18.00）。有免费档。AA 折算价为 Flash $16.5/百万字符，Flash-Lite $11/百万字符。
- **榜单**（[AA Provider](https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice)、[AA Controlled](https://artificialanalysis.ai/text-to-speech/leaderboard/controlled-voice)，2026-10-14）：Provider Voice 上 Flash 第 5（1274）、Flash-Lite 第 8（1248）；Controlled Voice 上 Flash-Lite 第 13（1058±14）、Flash 第 18（1048±14），见 §2.3。

#### 3.A.5 Inworld Realtime TTS-2 / TTS-2 Flash
- **发布方 / 时间**：Inworld AI。2026-05 发布研究预览，2026-08-31 GA（[官方博客](https://inworld.ai/blog/realtime-tts-2)）。
- **特点**：把多轮对话中的**前文音频**作为输入（“Conversational Awareness”）；支持自然语言 voice direction；同一音色覆盖 100+ 种语言，并可在一句话中切换语言；提供 Expressive / Balanced / Stable 三种稳定度模式。
- **克隆**：需要 5~15 秒参考音频。
- **延迟**：官方称 TTS 层中位首包 sub-200ms；Realtime API 兼容 OpenAI Realtime 协议。
- **价格**：按字符计费。AA 标注 TTS-2 $20.8/百万字符，TTS-2 Flash $10.4/百万字符。
- **榜单**：AA Provider 第 7（1260），Controlled 第 6（1140）。TTS Arena V2 上 Inworld TTS MAX 为 1558±21（API rank 2，tier S，见 §2.4）。

#### 3.A.6 MiniMax Speech 2.8（HD / Turbo）
- **发布方 / 时间**：MiniMax，2026-01-23（[官方新闻](https://www.minimax.io/news/minimax-speech-28)）。
- **特点**：原生语气 / 声音标签（`(laughs)`、`(breath)`、`(chuckle)` 等），10 秒克隆，优化了中日跨语言的“口音串味”问题。中文是 MiniMax 的强项，但 2.8 的中文客观指标本次**没有找到官方数字**。旧版 MiniMax-Speech 在 Seed-TTS-eval 上的数字为 zh CER 0.83 / en WER 1.65（见 §4）。
- **延迟**：本次没有核实到 2.8 的官方延迟数字。
- **价格**：AA 标注 HD $100、Turbo $60（每百万字符）。
- **榜单**：AA Provider：2.8 HD 1175、2.8 Turbo 1152；TTS Arena V2：2.8 HD 1540。

#### 3.A.7 字节跳动 豆包语音合成模型 2.0（Doubao-Seed-TTS 2.0）/ 声音复刻模型 2.0
- **发布方 / 时间**：字节跳动 / 火山引擎，2025-10-16（[IT之家](https://www.ithome.com/0/889/888.htm)）。
- **特点**：基于豆包大语言模型的新架构，具备语义理解和上下文推理能力，可以用自然语言调节语速、情绪、声线、音调和风格。官方称中小学全学科复杂公式朗读准确率达 90%。
- **克隆**：单独提供“声音复刻模型 2.0（Doubao-Seed-ICL 2.0）”。
- **价格（资源包，[火山引擎计费说明](https://www.volcengine.com/docs/6561/1359370)）**：语音合成 2.0 与声音复刻 2.0 定价相同：10 万字 28 元（2.8 元/万字符）、2000 万字 5400 元（2.7）、2 亿字 48000 元（2.4）、20 亿字 42 万元（2.1 元/万字符）。
- **榜单**：豆包 TTS 2.0 **没有出现在**本次抓取的 AA / TTS Arena 榜单里。Seed-TTS（2024 年论文版）只作为 Seed-TTS-eval 的参考系出现（zh CER 1.12 / en WER 2.25）。延迟数字本次**未核实**。

#### 3.A.8 阶跃星辰 StepAudio 2.5 TTS
- **发布方 / 时间**：阶跃星辰，2026-04-16（[IT之家](https://www.ithome.com/0/939/919.htm)）。
- **AA 条目与版本**：AA Provider 榜上的条目名为 “StepAudio 2.5 TTS (Aug 2026)”，发布时间标为 2026-08，Elo 1211。AA Controlled 榜上的条目名为 “StepAudio 2.5 TTS”，发布时间标为 2026-04，Elo 1074。也就是说，**Provider 榜的分数对应的是 AA 标注的 8 月版本**；它和 4-16 发布的版本是什么关系（是不是后续更新），本次**未核实**。
- **特点**：Global Context（全局语境）+ Inline Context（句中语境）双层控制，零样本克隆，可以控制情绪、节奏、停顿和潜台词，强调“演”文本而不只是“念”文本。
- **价格**：AA 标注 $85/百万字符（偏贵）；官方人民币价格**未核实**。

#### 3.A.9 其他值得关注的商业模型（简述）
- **HeyGen Voice**（HeyGen，2026-10-09）：一手来源为 HeyGen 通过 PRNewswire 发布的新闻稿（[转载于 BusinessPressWire](https://businesspresswire.com/2026/10/09/heygen-launches-heygen-voice-debuting-at-1-on-artificial-analysis-leaderboard/)）和 [HeyGen 开发者文档](https://developers.heygen.com/docs/voices/heygen-voice-instant-clone)。
  - 新闻稿要点：HeyGen 自研的语音模型，“在 HeyGen 平台和 API 内免费使用”；Professional Voice Clone 是 $99/月的附加服务，需要本人明确同意，用 30 分钟到 3 小时的语音训练；“Authentic Delivery”一项称能做出自然的强调、情感、角色感和节奏（厂商自述，没有参数级的情感控制说明）。
  - 语言：Instant Voice Clone 文档的 Languages 一节列出 39 个基础语言代码（ar、be、bg、ca、cs、da、de、el、en、es、fa、fi、fr、he、hi、hr、hu、id、it、ja、ko、mk、ms、nl、pl、pt、ro、ru、sk、sl、sr、sv、ta、th、tl、tr、uk、vi、zh），本文按原文计数。
  - 文档要点：Instant Voice Clone 只需一段录音（只取前 3 分钟）；`POST /v3/models/audio/tts` 返回 44.1kHz WAV，`/tts/stream` 通过 SSE 流式返回；模型名为 `heygen-voice-1`；文档列出的语言包含 `zh`。
  - 榜单与价格：AA Controlled Voice 第 1（1195），AA 标价 $30/百万字符。这与新闻稿说的“平台和 API 内免费”**口径不同**，计费细节未核实。AA 推文称其 Pronunciation Robustness 得分 83.1%，排第 10/29（[X](https://x.com/ArtificialAnlys/status/2108608264317599814)）。
- **Speechify Simba 3.2**（AA 标注 2026-07）：AA Provider 1245（第 9），Controlled 1088（第 9），标价 $6.6/百万字符，**是 AA Provider 前 10 名里最便宜的**。本次没有抓取 Speechify 的一手资料，语言、克隆、延迟等都**未核实**。
- **Fish Audio S2.1 Pro**（API）：AA 1192，$15。
- OpenAI GPT-Realtime-2（AA 1079，$191.6）、Microsoft MAI-Voice-1（AA 1033）、Xiaomi MiMo-V2.5-TTS（AA 1056，发布日期 AA 标为 “-”）：本文未展开。

### 3.B 开源 / 开放权重

#### 3.B.1 Fish Audio Drama 3（Preview）
- **发布方 / 时间**：Fish Audio。官方博客日期为 2026-10-13，权重发布在 HF `fishaudio/drama-3`（[博客](https://fish.audio/blog/fish-audio-drama-3/)、[GitHub](https://github.com/fishaudio/fish-speech)）。
- **架构**：35B MoE（每个 token 约 3B 激活），由 Qwen3.5-35B-A3B 初始化；codebook-MTP 解码器；输出 44.1kHz；RL 阶段只用人类偏好奖励模型。官方称单路 H200 解码 271 帧/秒（S2 为 177）。
- **语言**：83 种。导演指令可以用任何语言写，博客里有用中文指令驱动英文台词的示例。**官方没有公布 Drama 3 的分语种等级**：S2 Pro 卡片把中文列为 Tier 1，但不能直接套用到 Drama 3，中文质量未核实。
- **克隆 / 情感**：支持克隆、多角色场景、自然语言方括号指令、局部（成对标签）控制；编辑和 inpainting 功能目前仅限邀请测试。
- **许可证**：**Fish Audio Community License v1.0**。研究和非商用免费；本人及关联方年营收低于 50 万美元时可免费商用；超过这个门槛，或者要对外提供 TTS、克隆、推理服务，需要另行取得授权。
- **价格（API）**：标价 $50/百万字符，10-13 至 11-12 推广期 $10。AA 表中标为 $15，与官方不一致，**以官方为准**。
- **榜单**：AA 开放权重榜第 1（1243），Provider 总榜第 10；Controlled 第 14（1058）。AA Pronunciation Robustness 93.2%，全榜第 1（官方博客 + [AA 推文](https://x.com/ArtificialAnlys/status/2110081938140582085)）。
- **流式**：README 写明长文本支持流式，推荐用 SGLang-Omni 部署；首包延迟**未核实**。

#### 3.B.2 Fish Audio S2 Pro
- **发布方 / 时间**：Fish Audio。HF 仓库创建于 2026-03-09，技术报告为 [arXiv 2603.08823](https://arxiv.org/abs/2603.08823)（[HF](https://huggingface.co/fishaudio/s2-pro)）。
- **规格**：Dual-AR 架构，Slow AR 4B + Fast AR 0.4B（HF 显示模型总大小约 5B）；训练数据 1000 万小时以上，80+ 种语言（中、英、日为 Tier 1）；自由形式的 `[tag]` 指令，官方称支持 15,000+ 种标签。
- **延迟**：单张 H200 上 RTF 0.195，首包约 100ms（官方）。
- **许可证**：Fish Audio Research License，**商用需另行授权**。
- **指标**：Seed-TTS-eval 上 en WER 0.99、zh CER 0.54、hard 5.99。这组数字转引自 VoxCPM README，该表把模型写作 “FishAudio S2, 4B”，本文认为指的是同一个模型（S2 Pro 的 Slow AR 部分为 4B）。CV3-eval zh 2.65 / en 2.43（同一来源）。AA 开放权重榜 1118。

#### 3.B.3 Breeze TTS 2（BreezeBlue）
- **时间**：权重发布于 2026-08-25（[HF 卡片](https://huggingface.co/BreezeBlue/Breeze-TTS-2)）。
- **语言**：中英双语。
- **能力**：克隆（需要参考文本）、Voice Design、Voice Direction，以及中英文内联声音事件（`[笑]`、`[叹气]`）。
- **延迟**：H100 warm-up 后的快速路径下 TTFA < 40ms，RTF 0.32；显存约 7.7GiB（官方）。
- **许可证**：代码 Apache-2.0；**权重、衍生模型和自托管输出仅限研究 / 非商用**。AA 上另有官方 API，标价 $34/百万字符。
- **榜单**：AA 开放权重第 2（1221）；Controlled 1017。HF Open TTS 英文 WER 1.72。

#### 3.B.4 Qwen3-TTS（阿里通义，开源版）
- **时间**：2026-01-22 发布 0.6B / 1.7B 系列（[GitHub README](https://github.com/QwenLM/Qwen3-TTS)，HF 仓库创建于 2026-01-21）。
- **许可证**：Apache-2.0（HF 标签）。
- **语言**：10 种（中、英、日、韩、德、法、俄、葡、西、意），另有北京话、四川话等方言音色。
- **能力**：Base 模型 3 秒克隆；CustomVoice 有 9 个精品音色并支持指令控制风格；VoiceDesign 可以用自然语言描述来设计音色；支持“先设计、再克隆”的工作流。
- **延迟**：Dual-Track 流式架构，官方称输入单个字符后即可输出首包，端到端延迟最低 97ms（官方）。HF Open TTS 实测：Photon 引擎在 H200 上 TTFA 18.3ms（第三方转述）。另有 vLLM-Omni day-0 支持（README 注明当时只支持离线推理）。
- **指标**：Seed-TTS-eval 自报 12Hz-1.7B-Base 的 zh CER 0.77 / en WER 1.24，12Hz-0.6B-Base 为 0.92 / 1.32。VoxCPM README 转引为 zh 1.22 / en 1.23 / hard 6.76、SIM 77.0 / 71.7（**两处数字不一致**，可能是复现条件不同）。HF Open TTS 9 语言榜上，它是排名最高的 Apache-2.0 模型（第 3）。

#### 3.B.5 Fun-CosyVoice 3.0（阿里通义 FunAudioLLM）
- **时间**：2025-12 发布 Fun-CosyVoice3-0.5B-2512（Base + RL 版本）（[GitHub](https://github.com/FunAudioLLM/CosyVoice)）。
- **许可证**：Apache-2.0（HF 标签）。
- **语言**：9 种 + 18 种以上中文方言 / 口音（粤、闽南、川、东北、上海等），支持多语混说。
- **能力**：零样本克隆；指令可以控制语言、方言、情绪、语速、音量。
- **延迟**：双向流式（文本输入流式 + 音频输出流式），官方称延迟低至 150ms。
- **指标**（README 自报，Seed-TTS-eval）：0.5B-2512 为 zh 1.21 / en 2.24 / hard 6.71；**RL 版为 zh 0.81 / en 1.68 / hard 5.44**，SIM 77.4 / 69.5 / 75.0。HF Open TTS 英文 WER 1.68（第 6）。

#### 3.B.6 VoxCPM2（OpenBMB / 面壁智能）
- **时间**：2026-04 发布（[GitHub](https://github.com/OpenBMB/VoxCPM)）。之前有 VoxCPM1.5（2025-12）和 VoxCPM-0.5B（2025-09）。
- **许可证**：Apache-2.0，可商用。
- **规格**：2B 参数（MiniCPM-4 底座），训练数据 200 万小时以上，**30 种语言 + 9 种中文方言**（川、粤、吴、东北、河南、陕西、山东、天津、闽南），不需要语言标签；48kHz 输出。
- **能力**：Voice Design、可控克隆（克隆的同时用文字控制风格）、“极致克隆”（参考音频 + 文本续写），支持 LoRA / SFT 微调。
- **延迟**：RTX 4090 上 RTF 约 0.30，用 Nano-vLLM / vLLM-Omni 约 0.13；显存约 8GB；支持流式 API；也有 llama.cpp GGUF 版本（M4 Pro 上 RTF 约 1.76，非实时）。
- **指标**（README 自报）：Seed-TTS-eval 上 en WER 1.84 / SIM 75.3、zh CER 0.97 / SIM 79.5、hard 8.13 / 75.3；30 语种内部测试平均 WER 1.68%（用 Gemini 3.1 Flash Lite 做 ASR）。HF Open TTS 9 语言克隆视图中 SIM 最高（75.19，第三方转述）。

#### 3.B.7 IndexTTS-2 / IndexTTS-2.5（B 站 Index 团队）
- **时间**：IndexTTS-2 发布于 2025-09-08（**窗口外**，作为对照）；**IndexTTS-2.5 发布于 2026-08-10**（[GitHub](https://github.com/index-tts/index-tts)）。IndexTTS2 的参数量为 1.5B（据 CosyVoice / VoxCPM README 的对比表）；IndexTTS2.5 为 0.8B（据 IndexTTS README 的 CV3 表）。
- **语言**：2.5 支持中、英、日、西、阿。
- **能力**：音色与情感解耦。情感可以通过情感参考音频、8 维情感向量、从文本自动推断，或文字情感描述来控制，`emo_alpha` 调节强度。
- **时长 / 语速**：IndexTTS-2 的论文标题强调“精确合成时长控制”，但 README 注明这项功能“在该版本中尚未启用”。**IndexTTS-2.5 提供的是语速控制 `duration_factor`（0.5x~2.0x 时长）**，并改进了中文拼音、英文 CMU 音素、日文假名的可控性；README 还写明可以用 vLLM 做生产部署。
- **许可证**：bilibili Model Use License Agreement。据第三方 creativeainews 的解读，IndexTTS 2.5 在达到一定规模（1 亿月活或 10 亿元营收）之前可以商用；**原文条款请以 LICENSE 为准**。商务联系 indexspeech@bilibili.com。
- **指标**：IndexTTS2 在 Seed-TTS-eval 上为 zh 1.03 / en 2.23 / hard 7.12（CosyVoice / VoxCPM README 转引）。IndexTTS2.5-RL 在 CV3-eval 上为 zh 3.93 / en 3.89，5 语种平均 WER 6.00、SS 73.63；**跨语种（中文 prompt → 其他语言）平均 6.17**，在表中最优（README 自报）。
- **延迟**：README 称 2.5 比 2 推理更快，**没有给出毫秒数**。

#### 3.B.8 OmniVoice（k2-fsa / 新一代 Kaldi 团队）
- **时间**：HF 仓库创建于 2026-03-30，论文 [arXiv 2604.00688](https://arxiv.org/abs/2604.00688)（[GitHub](https://github.com/k2-fsa/OmniVoice)）。
- **规格**：扩散语言模型式架构，约 0.6~0.8B，**600+ 种语言**，RTF 最低 0.025。
- **能力**：克隆（最稳定的模式）、Voice Design（只用中英文数据训练）、`[laughter]` 等非语言符号、拼音 / 音素纠音。
- **许可证**：代码 Apache-2.0；**权重 CC-BY-NC**（受 Emilia 等训练数据约束）。
- **指标**：HF Open TTS 9 语言默认音色榜和克隆榜均为第 1（3.25 / 3.52，第三方转述）；CV3-eval zh 3.41 / en 3.62（IndexTTS README 转引）。

#### 3.B.9 Higgs TTS 3（Boson AI）
- **时间**：HF 仓库创建于 2026-06-04；AA 上标为 “Higgs Audio V3 TTS（Jun 2026）”（[HF](https://huggingface.co/bosonai/higgs-tts-3-4b)）。
- **规格**：约 4B 自回归模型，24kHz；官方称在 102 种语言上 WER/CER 达到个位数。
- **能力**：零样本克隆；内联 `<|emotion:…|>`、`<|style:…|>`、`<|prosody:…|>`、`<|sfx:…|>` 控制，情绪类别很多；支持 SSE 流式，首包“亚秒级”。
- **许可证**：研究 / 非商用。另有 **Creator Use Grant**：创作者可以免费制作并变现播客、视频等内容（需要注明 Boson AI Higgs Audio），但不能用于托管服务或嵌入产品。
- **指标（Boson 自报，在统一条件下重跑）**：卡片说明表中数字为“各基准语言集合上的宏平均 WER/CER（×100）”，并且是用原始指标和归一化端到端复现的结果。SeedTTS 一行：Higgs TTS 3 1.11、OmniVoice 1.21、Qwen3-TTS-1.7B 1.30、Fish Audio S2 Pro 1.31、IndexTTS-2 1.63、FireRedTTS-2 1.72；CV3 一行 Higgs TTS 3 为 4.41。AA 开放权重榜 1039。
- **为什么和 §4 的数字对不上**：§4 是各模型作者自己报告的分拆分数字（zh 和 en 分开）。如果简单把 zh 和 en 做算术平均，Qwen3-TTS 自报是 (0.77+1.24)/2≈1.01，Fish S2 转引是 (0.54+0.99)/2≈0.77，都低于 Higgs 卡片里的 1.30 / 1.31。可能的原因有：Boson 自己重跑了推理（采样随机性、参考音频处理方式不同）、ASR 和文本归一化不同、“语言集合”是否包含 test-hard 不明确等。卡片**没有逐项说明**这些细节，差异的具体来源未核实。所以 Higgs 表里的数字只适合在 Higgs 表内部互相比较，不宜和其他 README 的数字混在一起比。

#### 3.B.10 Voxtral TTS（Mistral AI）
- **时间**：2026-03（模型名 4B-TTS-2603，论文 [arXiv 2603.25551](https://arxiv.org/abs/2603.25551)；[HF](https://huggingface.co/mistralai/Voxtral-4B-TTS-2603)）。
- **语言**：9 种（英、法、西、德、意、葡、荷、阿、印地），**不含中文**。
- **能力**：20 个预置音色，可以适配新音色（权重内的参考音色为 CC BY-NC）。官方称在 9 种主要语言中具备“natural prosody and emotional range”（自然韵律和情感表达范围）（厂商自述）。
- **延迟**：单张 H200、vLLM-Omni、并发 1 时，延迟 70ms、RTF 0.103（官方）。
- **许可证**：**CC-BY-NC-4.0**。AA 标注其 API 价为 $16/百万字符。AA 开放权重榜 1088。本次**没有找到** Voxtral TTS 的 WER/CER 客观指标。

#### 3.B.11 Step-Audio-EditX（阶跃星辰）
- **时间**：HF 仓库创建于 2025-10-29；2025-11-19 发布新版（多音字控制），2025-11-28 增加日、韩语（[HF](https://huggingface.co/stepfun-ai/Step-Audio-EditX)）。
- **定位**：3B 参数的 LLM 音频**编辑**模型（经过 RL），可以对情绪、说话风格、副语言做迭代编辑，也能克隆；支持四川话、粤语标签。
- **许可证**：代码 Apache-2.0；**权重许可证本次未单独核实**。AA 开放权重榜 1097（Mar 2026 版）。

#### 3.B.12 Chatterbox 家族（Resemble AI）
- **型号与时间**：
  - Chatterbox 原版：HF 仓库创建于 2025-04-24，AA 标为 2025-05（**窗口外**，作为对照）。
  - Chatterbox-Turbo：350M，英文，HF 仓库创建于 2025-12-02。
  - Chatterbox-Nano：110M，英文，8 核 CPU 上约 3 倍实时；发布日期**未核实**。
  - **Chatterbox Multilingual V3**：500M，23+ 种语言，支持语言列表中包含中文；发布日期**未核实**（[GitHub](https://github.com/resemble-ai/chatterbox)）。
- **能力**：零样本克隆；`[laugh]`、`[cough]` 等副语言标签；exaggeration / cfg 控制夸张度；所有输出内置 PerTh 隐形水印。
- **许可证**：MIT。
- **指标**：AA 上的 Chatterbox（原版）为 1026；HF Open TTS 英文 WER 1.70。Multilingual V3 的独立评测本次**没有找到**。

#### 3.B.13 VibeVoice（微软）
- **型号**：
  - VibeVoice-TTS 1.5B：2025-08-25 开源（**窗口外**，作为对照），可生成最长 90 分钟、最多 4 位说话人的长篇对话，README 称支持英文、中文等语言。
  - **VibeVoice-Realtime-0.5B**：2025-12-03 开源，支持流式文本输入，首个可听音频约 300ms。2025-12-16 增加了实验性音色，覆盖 9 种语言（德、法、意、日、韩、荷、波、葡、西）和 11 种英文风格（[GitHub](https://github.com/microsoft/VibeVoice)）。
- **许可证**：MIT。但官方明确表示“不建议在未经进一步测试的情况下用于商业或真实场景”；2025-09 曾因滥用问题调整过仓库。
- **指标**：VibeVoice-1.5B 在 Seed-TTS-eval 上为 zh 1.16 / SIM 74.4、en 3.04 / SIM 68.9。这组数字在 VoxCPM README 和 CosyVoice README 中**一致**，两处都有。Realtime-0.5B 为 en 2.05 / SIM 63.3，只见于 CosyVoice README。AA 上 VibeVoice 1.5B 为 957。

#### 3.B.14 Kyutai Pocket TTS
- **时间**：技术报告 / 博客发布于 2026-01-13（[kyutai.org/blog/2026-01-13-pocket-tts](https://kyutai.org/blog/2026-01-13-pocket-tts)）；**2026-08 开源训练代码**（[GitHub](https://github.com/kyutai-labs/pocket-tts)，X 上也有讨论，如 [@daivbr](https://x.com/daivbr/status/2092466396143722501)）。
- **规格**：100M 参数（生成模型 90M + 编解码器解码器 10M），只用 2 个 CPU 核心，首包约 200ms，在 M4 MacBook Air 上约 6 倍实时；支持流式和克隆（官方博客称约需 5 秒参考音频；克隆权重需要在 HF 上同意条款），可在浏览器端运行。
- **语言**：1 月发布时只用公开英文数据（8.8 万小时）训练；当前 README 列出英、法、德、葡、意、西、荷 7 种（**不含中文**）。
- **指标**：博客自报 LibriSpeech test-clean WER 1.84（Kokoro 1.93、F5-TTS 2.21、Chatterbox Turbo 3.24）。HF Open TTS 英文 WER 1.62，CPU TTFA 272.7ms（第三方转述）。
- **许可证**：**来源之间不一致**。1 月博客写明“以 MIT 许可开源 Pocket TTS”，而 HF 模型仓库 `kyutai/pocket-tts` 的许可证标签为 CC-BY-4.0（第三方 creativeainews 也按 CC-BY-4.0 解读，需要署名）。推测代码与权重可能分别授权，但**未核实**；使用前请查看仓库里的 LICENSE 和 voices 许可页面。

#### 3.B.15 Supertonic 3（Supertone）
- **时间**：HF 仓库创建于 2026-05-06（[HF](https://huggingface.co/Supertone/supertonic-3)）。
- **定位**：端侧 TTS，约 0.1B，31 种语言（从 v2 的 5 种扩展而来）。按 HF 卡片的 Supported Languages 表，支持英、韩、日、阿、德、西、法、印地、意、荷、波、葡、俄、越等，**不含中文**。
- **许可证**：模型 OpenRAIL-M（有使用限制），示例代码 MIT。
- **指标**：HF Open TTS 英文 WER 1.54（第 2）。但**没有流式模式**，单句 TTFA 为 1918.6ms（第三方转述），不适合实时对话。

#### 3.B.16 NVIDIA Magpie TTS Multilingual 357M
- **时间**：HF 仓库创建于 2025-12-11；2026-07-21 发布 v2607，新增阿、韩、葡（[HF](https://huggingface.co/nvidia/magpie_tts_multilingual_357m)）。
- **语言**：12 种，含中文；5 个固定音色，**不用于零样本克隆**。
- **许可证**：NVIDIA Open Model License，卡片注明“ready for commercial use”。AA 1065。

#### 3.B.17 Kokoro-82M（hexgrad，基线，**窗口外**）
- v1.0 于 2025-01-27 发布，**不在近一年窗口内**，作为轻量基线收录（[HF](https://huggingface.co/hexgrad/Kokoro-82M)）。
- 规格：Apache-2.0，82M 参数，可在 CPU 上运行。v1.0 共 8 种语言、54 个音色，其中普通话有 8 个音色（4 女 4 男，见 [VOICES.md](https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md)）。普通话训练时长未公布，中文质量未核实。
- 不支持克隆。
- 榜单与价格：AA 1063，表格显示 $0.7，FAQ 精确值为 $0.65/百万字符，全榜最便宜；HF Open TTS 英文 WER 第 1（1.46）。

#### 3.B.18 Raon-OpenTTS（KRAFTON）
- 论文 [arXiv 2605.20830](https://arxiv.org/abs/2605.20830)，**v1 提交于 2026-05-20**（arXiv 摘要页）。代码和权重的许可证为 Apache-2.0（[GitHub](https://github.com/krafton-ai/RAON-OpenTTS)）。
- **开放权重 + 开放数据**：数据池 61.5 万小时，筛选后训练集 51 万小时；参数 0.3B~1B。
- 指标：README 自报 Seed-TTS-eval（EN）WER 1.78 / SIM 0.749。
- 克隆：README 没有专门的“voice cloning”功能说明。但它在 Seed-TTS-Eval 的零样本、cross-sentence prompt 设置下报告了说话人相似度 SIM，并写明推理时以参考音频（原始、未裁剪）作为条件信号，所以属于**参考音频条件的零样本克隆**。克隆质量只有上面这一组 SIM 可参考。
- 只支持英文。AA 上另有 “Raon SpeechLM”（2026-04，Provider 1005），它和 Raon-OpenTTS 的关系**未核实**。

---

## 4. Seed-TTS-eval 客观指标汇总（WER/CER 越低越好，SIM 越高越好）

> ⚠️ 下表数字都来自各模型作者的 README，或者别的项目 README 里的转引，**不是同一方在统一条件下复现的结果**。每一行的“来源”列标明出处；同一模型如果有多个来源，就分行列出、不混在一行。例如 CosyVoice 3 在 Qwen3-TTS README 中为 zh 0.71 / en 1.45，在 VoxCPM README 中 1.5B 版为 1.12 / 2.22。这张表只适合判断大致档位。

| 模型 | 参数 | 开源 / 许可 | test-zh CER↓ | zh SIM↑ | test-en WER↓ | en SIM↑ | test-hard CER↓ | hard SIM↑ | 来源 |
|---|---|---|---|---|---|---|---|---|---|
| Seed-TTS（闭源参照，2024，窗口外） | - | ✗ | 1.12 | 79.6 | 2.25 | 76.2 | 7.59 | 77.6 | VoxCPM README |
| MiniMax-Speech（旧版，窗口外） | - | ✗ | 0.83 | 78.3 | 1.65 | 69.2 | - | - | VoxCPM README |
| Fish Audio S2（VoxCPM 表中写作 “FishAudio S2, 4B”，即 §3.B.2 的 S2 Pro：Slow AR 4B + Fast AR 0.4B） | 4B+0.4B | 权重（研究许可） | **0.54** | - | **0.99** | - | 5.99 | - | VoxCPM README（转引） |
| Qwen3-TTS-12Hz-1.7B-Base（自报） | 1.7B | ✓ Apache | 0.77 | 未报告 | 1.24 | 未报告 | 未报告 | 未报告 | Qwen3-TTS README |
| Qwen3-TTS 1.7B（转引） | 1.7B | ✓ Apache | 1.22 | 77.0 | 1.23 | 71.7 | 6.76 | 74.8 | VoxCPM README |
| Qwen3-TTS-12Hz-0.6B-Base（自报） | 0.6B | ✓ Apache | 0.92 | 未报告 | 1.32 | 未报告 | 未报告 | 未报告 | Qwen3-TTS README |
| Fun-CosyVoice3-0.5B-2512_RL | 0.5B | ✓ Apache | 0.81 | 77.4 | 1.68 | 69.5 | **5.44** | 75.0 | CosyVoice README |
| Fun-CosyVoice3-0.5B-2512 | 0.5B | ✓ Apache | 1.21 | 78.0 | 2.24 | 71.8 | 6.71 | 75.8 | CosyVoice README |
| VoxCPM2 | 2B | ✓ Apache | 0.97 | **79.5** | 1.84 | **75.3** | 8.13 | 75.3 | VoxCPM README |
| IndexTTS2（2025-09，窗口外） | 1.5B | ✓ bilibili 许可 | 1.03 | 76.5 | 2.23 | 70.6 | 7.12 | 75.5 | VoxCPM / CosyVoice README（两处一致） |
| VibeVoice-1.5B（2025-08，窗口外） | 1.5B | ✓ MIT | 1.16 | 74.4 | 3.04 | 68.9 | - | - | VoxCPM / CosyVoice README（两处一致） |
| VibeVoice-Realtime-0.5B | 0.5B | ✓ MIT | - | - | 2.05 | 63.3 | - | - | CosyVoice README |
| Raon-OpenTTS-1B | 1B | ✓ Apache | - | - | 1.78 | 74.9（原文 0.749） | - | - | Raon README |

说明：
- **Higgs TTS 3 没有放进上表**。它的卡片只给出 SeedTTS 的“语言集合宏平均 WER/CER”（1.11），没有分开报 zh 和 en，而且是 Boson 自己在统一条件下重跑的结果，和上表口径不同，详见 §3.B.9。
- VoxCPM README 的表里还有 LongCat-Audio-DiT 等模型。本报告没有核实它们的发布方和发布时间，正文也没有对应章节，所以没有列入。
- SIM 一律换算成百分数（×100）。
- **粗体 = 不计闭源参照行（Seed-TTS、MiniMax-Speech）时的列内最优**。闭源 Seed-TTS 的 SIM（zh 79.6 / en 76.2 / hard 77.6）其实高于表中所有开源模型。

CV3-eval 多语种（IndexTTS README 自报，WER↓ / SS↑，5 语种平均，阿语为内部测试集）：Fish S2 Pro 5.94 / 64.49；IndexTTS2.5-RL 6.00 / 73.63；OmniVoice 6.76 / 70.39；VoxCPM2 7.22 / 72.02。

---

## 5. 综合对比表

| 模型 | 发布方 | 发布时间 | 开源 / 商业 | 中文 | 英文 | 多语种 | 克隆 | 情感 / 指令控制 | 延迟 / 流式（多为自报） | 价格 / 许可 | 代表性榜单 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Eleven v4 / v4 Turbo | ElevenLabs | 2026-09-28 | 商业 | 支持（官方称普通话、粤语提升大；无中文盲测） | ★★★★★ | 90+ | 10 秒 IVC + PVC | 可叠加的内联音频标签；多说话人；context stitching | Turbo 推理约 100ms，首声约 150ms，双向流式 | $40（Turbo）/ $80 每百万字符（AA） | AA Provider #1/#2 |
| Qwen-Audio-3.1-TTS-Plus / Flash | 阿里 | 2026-09-23 | 商业 | 支持，多方言（具体列表未核实） | 支持 | 多语种 + 跨语言音色迁移 | ✓（Flash 对噪声参考更鲁棒） | free-style 指令 + 标签 | Flash 支持 WebSocket 流式（无毫秒数） | Plus $19.3（AA，官方价未核实）；Flash 输出 12 元/百万 Token；3.0-Plus 1.4 元/万字符 | AA Provider #3、Controlled #2 |
| Sonic 3.6 | Cartesia | 2026-08-27 | 商业 | 无公开数据 | 强 | 44 种语言 / 61 locale | IVC | 随上下文调整情感（官方自述） | 首包 <90ms | $49（AA） | AA Provider #4 |
| Gemini 3.8 Flash TTS / Flash-Lite | Google | 2026-09-23 | 商业 | 支持（100+ 种语言之一） | 强 | 100+ | 30 秒复刻（需同意录音） | 自然语言导演、双人对话；官方称可保持数小时长音频稳定 | 无数字 | 输入 $0.5 / 输出 $9 每百万 token（2027 起翻倍）；AA 折算 $16.5 / $11 | AA Provider #5 / #8；Controlled #18 / #13 |
| Realtime TTS-2 / TTS-2 Flash | Inworld | 2026-08-31 GA | 商业 | 支持（100+ 种语言之一） | 强 | 100+ | 5~15 秒 | 自然语言方向 + 感知前文音频 | 中位首包 <200ms | $20.8 / $10.4（AA） | AA Provider #7 / #13 |
| HeyGen Voice | HeyGen | 2026-10-09 | 商业 | 文档语言列表含 zh | 支持 | 39 种基础语言代码（文档） | 单段录音 IVC；PVC 需 30 分钟~3 小时 | 新闻稿称有自然的强调、情感、角色感和节奏（厂商自述） | 有 SSE 流式接口（无毫秒数） | 新闻稿称平台和 API 内免费；AA 标 $30；PVC 附加 $99/月 | AA Controlled #1（1195） |
| Simba 3.2 | Speechify | 2026-07（AA） | 商业 | 未核实 | 未核实 | 未核实 | 未核实 | 未核实 | 未核实 | $6.6（AA） | AA Provider #9、Controlled #9 |
| Speech 2.8 HD / Turbo | MiniMax | 2026-01-23 | 商业 | 支持（优化了中日跨语言） | 强 | 多语种 | 10 秒 | 语气标签 | 未核实 | $100 / $60（AA） | AA 1175；Arena V2 1540 |
| 豆包语音合成 2.0 | 字节 / 火山引擎 | 2025-10-16 | 商业 | 支持（未上公开榜） | - | 未核实 | 声音复刻 2.0 | 自然语言控制；公式朗读准确率 90%（官方） | 未核实 | 2.1~2.8 元/万字符（资源包） | 未上榜 |
| StepAudio 2.5 TTS | 阶跃星辰 | 2026-04-16（AA 另有 2026-08 条目） | 商业 | 支持 | 支持 | 未核实 | 零样本 | 全局 + 句中语境 | 未核实 | $85（AA） | AA Provider 1211（8 月条目）/ Controlled 1074 |
| Fish Audio Drama 3 | Fish Audio | 2026-10-13 | 开放权重（社区许可） | 支持（分语种等级未公布） | 强 | 83 | ✓ | 自由形式方括号指令 + 局部标签 | 支持流式（首包未核实） | 年营收 <50 万美元可免费商用；API $50（推广价 $10） | AA 开放权重 #1（1243） |
| Fish Audio S2 Pro | Fish Audio | 2026-03 | 开放权重（研究许可） | Tier 1 | 强 | 80+ | ✓ | 15,000+ 种标签 | H200 上首包约 100ms | 商用需授权 | Seed zh 0.54 / en 0.99（转引） |
| Breeze TTS 2 | BreezeBlue | 2026-08-25 | 开放权重（非商用） | ✓ | ✓ | 仅中英 | ✓ | Voice Design / Direction / 声音事件 | H100 上 TTFA <40ms | 非商用；API $34 | AA 开放权重 #2 |
| Qwen3-TTS | 阿里通义 | 2026-01-22 | **Apache-2.0** | 强，含方言音色 | 强 | 10 | 3 秒 | 指令 + Voice Design | 首包最低 97ms | 免费商用 | Seed zh 0.77 / en 1.24（自报） |
| Fun-CosyVoice3-0.5B | 阿里通义 | 2025-12 | **Apache-2.0** | 强（18+ 方言） | 中上 | 9 | ✓ | 指令（方言、情绪、语速） | 双向流式约 150ms | 免费商用 | RL 版 zh 0.81 / hard 5.44 |
| VoxCPM2 | OpenBMB | 2026-04 | **Apache-2.0** | 强（9 种方言） | 强 | 30 | ✓ 可控克隆 | Voice Design + 风格指令 | 4090 上 RTF 0.13~0.30，支持流式 | 免费商用 | zh 0.97 / SIM 79.5 |
| IndexTTS-2.5 | B 站 | 2026-08-10 | 开源（bilibili 许可） | 强 | 强 | 5 | ✓ | 音色 / 情感解耦、情感向量；语速 `duration_factor` 0.5~2.0x | 无毫秒数（可用 vLLM 部署） | 有规模门槛（以原文为准） | CV3 跨语种平均 6.17，表中最优 |
| OmniVoice | k2-fsa | 2026-03/04 | 权重 CC-BY-NC | ✓ | ✓ | 600+ | ✓ | 属性设计、非语言符号 | RTF 0.025 | 非商用 | HF Open TTS 多语种 #1 |
| Higgs TTS 3 | Boson AI | 2026-06 | 非商用 + 创作者授权 | ✓ | ✓ | 102 | ✓ | 丰富的情绪 / 风格 / 韵律 token | SSE 流式，首包亚秒级 | 非商用 | SeedTTS 宏平均 1.11（Boson 重跑） |
| Voxtral TTS | Mistral | 2026-03 | CC-BY-NC-4.0 | ✗ | ✓ | 9 | 可适配新音色 | 官方称有自然韵律和情感范围 | H200 上 70ms | 非商用；API $16 | AA 1088 |
| Step-Audio-EditX | 阶跃星辰 | 2025-10~11 | 代码 Apache-2.0，权重许可未核实 | ✓（另有川、粤标签） | ✓ | +日、韩（2025-11-28） | ✓ | 情绪 / 风格 / 副语言的迭代编辑；多音字控制 | 未核实 | 未核实 | AA 开放权重 #4（1097） |
| Chatterbox Turbo / Nano / ML-V3 | Resemble AI | Turbo 2025-12；原版 2025-05（窗口外） | MIT | ML-V3 支持语言含中文 | 强 | 23+ | ✓ | 副语言标签 + exaggeration | Turbo 面向低延迟；Nano 可在 CPU 运行 | 免费商用（带水印） | AA 1026（原版） |
| VibeVoice-Realtime-0.5B | 微软 | 2025-12-03 | MIT（官方不建议商用） | 未核实 | ✓ | 9 种语言的实验性音色 | - | - | 首包约 300ms | 研究向 | en WER 2.05 |
| Pocket TTS | Kyutai | 2026-01-13（训练代码 2026-08） | 博客称 MIT / HF 标签 CC-BY-4.0（不一致） | ✗ | ✓ | 7 种欧洲语言 | ✓（约 5 秒，gated） | - | 2 核 CPU 首包约 200ms | 见许可列 | HF Open TTS en 1.62 |
| Supertonic 3 | Supertone | 2026-05 | OpenRAIL-M | ✗（31 种语言中不含中文） | ✓ | 31 | - | - | 无流式 | 有使用限制 | HF Open TTS en #2 |
| Magpie Multilingual 357M | NVIDIA | 2025-12 / 2026-07 | NVIDIA Open Model License | ✓ | ✓ | 12 | ✗ | 固定音色 | - | 可商用 | AA 1065 |
| Raon-OpenTTS-1B | KRAFTON | 2026-05-20（arXiv v1） | Apache-2.0（开放权重 + 开放数据） | ✗ | ✓ | 仅英文 | ✓（以参考音频为条件的零样本，见 §3.B.18） | - | 未核实 | 免费商用 | Seed-EN WER 1.78 / SIM 0.749 |
| Kokoro-82M（窗口外） | hexgrad | 2025-01-27 | Apache-2.0 | 8 个普通话音色（质量未核实） | 强 | 8 种语言 / 54 个音色 | ✗ | - | CPU 可运行 | 免费商用 | HF Open TTS en #1 |

（★ 和“强 / 中上”是根据上文榜单和自报数据做的定性判断，不是统一评测结果；“未核实”表示本次没有找到一手数据。）

---

## 6. 场景推荐

### 6.1 中文实时语音 Agent（客服、陪伴、语音助手）
- **首选 API**：**Qwen-Audio-3.1-TTS-Flash**。官方文档写明它面向实时交互，“支持多种语言和方言”和流式合成，按 token 计费（输出 12 元/百万 Token）。如果更看重音质，可以选 3.1-Plus（AA 总榜第 3、克隆榜第 2，AA 标价约为 Eleven v4 的 1/4）。
- **备选**：豆包语音合成 2.0（中文生态，资源包 2.1~2.8 元/万字符）、MiniMax Speech 2.8 Turbo。出海或多语种场景可选 **Eleven v4 Turbo**（首声约 150ms）、**Inworld Realtime TTS-2**（能感知前文音频，兼容 OpenAI Realtime 协议）、**Cartesia Sonic 3.6**（<90ms）。
- **自托管**：**Qwen3-TTS 1.7B / 0.6B**（Apache，首包最低 97ms；建议配合 Photon、vLLM-Omni 等优化引擎，同一份权重换引擎 TTFA 相差可达 17 倍），或 **Fun-CosyVoice3-0.5B**（双向流式约 150ms，方言多）。
- ⚠️ 这些延迟基本都是厂商在 H100/H200 上自测的。上线前请用自己的网络和硬件实测 P50/P95 首包延迟。

### 6.2 有声书 / 长篇内容 / 广播剧
- **商业**：
  - **Eleven v4**：官方称 context stitching 能让整本有声书像一次录完；支持多说话人和音效，重复生成不漂移（见 §3.A.1）。
  - **Gemini 3.8 Flash TTS**：官方称能在数小时连续音频中保持音色和节奏，支持双人剧本，价格低（见 §3.A.4）。
  - **Qwen-Audio-3.1-TTS-Next**：人声、音效、环境音一体生成，文档列出的语种为中文和英文。但它是**非流式**的，单次生成上限为播客 240 秒、其他场景 120 秒，长篇需要分段（见 §3.A.2）。
- **开放权重**：**Fish Audio Drama 3**（AA 开放权重第 1，发音鲁棒性第 1，面向“渲染一次、长期保留”的剧本内容；年营收低于 50 万美元的团队可免费商用）。
- **完全开源可商用**：**VoxCPM2**（48kHz，Voice Design，Apache）。**IndexTTS-2.5** 有情感解耦和语速调节（`duration_factor`），但精确时长控制在 IndexTTS-2 中尚未启用，配音对口型的效果需要实测；另外要注意许可门槛。

### 6.3 声音克隆
- **商业**：AA Controlled Voice 榜前两名是 **HeyGen Voice**（#1，1195，AA 标 $30）和 **Qwen-Audio-3.1-TTS-Plus**（#2，1180，AA 标 $19.3，与 #7 Qwen-Audio-3.0-TTS-Plus 的 $19.3 并列前 8 名中的最低价）。之后是 Eleven v4 / v4 Turbo（$80 / $40）。如果更在意单价，榜上更便宜的有 Simba 3.2（#9，$6.6）、Realtime TTS-2 Flash（#12，$10.4）、Gemini 3.8 Flash-Lite（#13，$11.0），但克隆盲听分明显更低。正规服务都要求音色所有者同意：Google 要求提交同意录音，ElevenLabs 要求验证，HeyGen 要求明确同意。
- **开源可商用**：**Qwen3-TTS Base**（3 秒克隆，HF Open TTS 克隆视图中排名最高的 Apache 模型）、**VoxCPM2**（9 语言克隆视图 SIM 最高，支持可控克隆）、**Fun-CosyVoice3**（中文 SIM 高）。
- **仅限研究**：OmniVoice、Fish S2 Pro、Higgs TTS 3 的克隆客观分很高，但许可受限。

### 6.4 端侧 / 离线 / CPU
- **英文和欧洲语言**：**Pocket TTS**（2 核 CPU，首包约 200ms，支持克隆，可在浏览器运行；许可证说法不一，见 §3.B.14）、**Kokoro-82M**（Apache，吞吐高，不支持克隆）、**Chatterbox-Nano**（MIT，110M）。
- **多语种（不含中文）**：**Supertonic 3**（31 种语言，但没有流式，OpenRAIL-M 有使用限制）。
- **含中文的较小模型**：
  - **Kokoro-82M**：带 8 个普通话音色，质量未核实，不支持克隆。
  - **NVIDIA Magpie TTS Multilingual 357M**：12 种语言含中文，NVIDIA Open Model License，卡片写明“ready for commercial use”；固定 5 个音色，不做零样本克隆。注意卡片说明它针对 NVIDIA GPU 设计和优化，CPU 或端侧表现未核实。
  - **Chatterbox Multilingual V3**：500M，MIT，23 种语言含中文（README 语言列表），支持零样本克隆，输出带水印；没有 CPU 或端侧数据（未核实）。
- **不算轻量**：VoxCPM2 有 GGUF 版本可以用 llama.cpp 本地运行，但它是 2B 模型，在 M4 Pro 上 RTF 约 1.76，即非实时，只适合离线批量生成。
- **结论**：**目前还没有经过公开验证、能在端侧做中文实时合成的高质量小模型**。建议对 Kokoro 普通话音色、Magpie 357M、Chatterbox Multilingual V3 以及量化后的 Qwen3-TTS-0.6B 做目标硬件实测。

### 6.5 低成本自托管（以商用合规为先）
- 按许可证友好程度排序：**Qwen3-TTS（Apache）≈ VoxCPM2（Apache）≈ Fun-CosyVoice3（Apache）> Chatterbox（MIT，带水印）> Magpie（NVIDIA 许可）**。只做英文的话，还可以考虑 Raon-OpenTTS（Apache，开放数据）。
- 单卡参考：VoxCPM2 约 8GB 显存（官方）；Breeze TTS 2 约 7.7GiB（官方，但非商用）；Qwen3-TTS、CosyVoice3 的官方显存数字本次未核实（CosyVoice3 只有 0.5B，参数量更小）。
- 不想自备 GPU 的话，可以直接用 API：Qwen-Audio-3.1-TTS-Flash（输出 12 元/百万 Token）、Gemini 3.8 Flash-Lite TTS（AA 折算 $11/百万字符）、Speechify Simba 3.2（$6.6/百万字符，AA Provider 第 9；能力细节未核实）。

### 6.6 多语种 / 出海
- **商业**：Eleven v4（90+）、Gemini 3.8（100+）、Inworld TTS-2（100+，同一音色跨语种）、Cartesia Sonic 3.6（44 种语言 / 61 locale，印度语种强）。
- **开源可商用**：**VoxCPM2（30 种语言）**、**Qwen3-TTS（10 种语言，9 语言客观榜上排名最高的 Apache 模型）**。
- **长尾语言（仅限研究）**：OmniVoice（600+）、Higgs TTS 3（102）、Fish S2 Pro / Drama 3（80+ / 83）。

---

## 7. 风险与注意事项

1. **榜单时效性**：AA 在 2026-10-14 的快照里，前 10 名中有 9 个是近两个月发布的。X 上各日期帖子引用的 Elo 数值都不一样（比如 Eleven v4 在 1319~1328 之间）。选型前请重新查看实时榜单。
2. **AA 榜单只有英语口音**（Controlled 榜明确是 4 个美音 + 4 个英音，Provider 榜筛选项只有 US/UK）。**中文质量不能直接用 AA 排名推断**，中文场景应以 Seed-TTS-eval（zh）、方言测试和自己的盲听为准。
3. **“开放权重” ≠ “可商用”**：Drama 3（营收门槛）、S2 Pro、Breeze TTS 2、OmniVoice、Voxtral、Higgs TTS 3 都有商用限制；IndexTTS 使用自定义许可证。
4. **价格口径不同**：AA 统一折算为“每百万字符美元价”，阿里、火山引擎是“元/万字符”或“元/百万 token”，Gemini 按 token 计费，而且 2027-01-01 起涨价一倍。另外，Fish Drama 3 的官方 API 价（$50）与 AA 标注（$15）不一致。
5. **合规**：克隆功能普遍要求同意验证；Gemini 自带 SynthID，Chatterbox 自带 PerTh 水印。在国内上线还需要遵守深度合成相关规定（本报告未展开）。

---

## 8. 来源清单（均为本次实际抓取；arXiv 链接取自官方 README、卡片或博客，Raon 的摘要页已直接抓取）

**榜单与评测**
- Artificial Analysis Provider Voice：https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice
- AA 开放权重：https://artificialanalysis.ai/text-to-speech/leaderboard/provider-voice/open-weights
- AA Controlled Voice：https://artificialanalysis.ai/text-to-speech/leaderboard/controlled-voice
- TTS Arena V2 API：https://tts-agi-tts-arena-v2.hf.space/api/leaderboard
- HF Open TTS Leaderboard 博客：https://huggingface.co/blog/open-tts-leaderboard
- creativeainews 对 Open TTS 榜的转述和许可证解读（第三方）：https://www.creativeainews.com/articles/open-tts-leaderboard-license-cpu-test-2026/
- Seed-TTS-eval：https://github.com/BytedanceSpeech/seed-tts-eval/

**商业模型**
- ElevenLabs v4 博客：https://elevenlabs.io/blog/eleven-v4 ；产品页：https://elevenlabs.io/v4 ；TechCrunch：https://techcrunch.com/2026/09/28/elevenlabs-new-v4-speech-model-supports-more-expression-control-and-90-languages/
- 阿里 Qwen-Audio-3.1 发布（IT之家）：https://www.ithome.com/1/006/280.htm
  - 3.0-TTS-Plus 文档：https://help.aliyun.com/zh/model-studio/qwen-audio-3-0-tts-plus
  - 3.1-TTS-Flash 文档：https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-flash
  - 3.1-TTS-Next 文档：https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-next
- Cartesia Sonic 3.6：https://www.cartesia.ai/blog/sonic-3.6
- Gemini 3.8 TTS：https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-8-text-to-speech/ ；定价：https://ai.google.dev/gemini-api/docs/pricing
- Inworld Realtime TTS-2：https://inworld.ai/blog/realtime-tts-2
- HeyGen Voice 新闻稿（HeyGen 经 PRNewswire 发布，BusinessPressWire 转载）：https://businesspresswire.com/2026/10/09/heygen-launches-heygen-voice-debuting-at-1-on-artificial-analysis-leaderboard/ ；开发者文档：https://developers.heygen.com/docs/voices/heygen-voice-instant-clone
- MiniMax Speech 2.8：https://www.minimax.io/news/minimax-speech-28
- 豆包语音合成 2.0（IT之家）：https://www.ithome.com/0/889/888.htm ；火山引擎计费：https://www.volcengine.com/docs/6561/1359370
- StepAudio 2.5 TTS（IT之家）：https://www.ithome.com/0/939/919.htm

**开源模型**
- Fish Audio Drama 3 博客：https://fish.audio/blog/fish-audio-drama-3/ ；fish-speech：https://github.com/fishaudio/fish-speech ；S2 Pro：https://huggingface.co/fishaudio/s2-pro ；S2 技术报告 [arXiv 2603.08823](https://arxiv.org/abs/2603.08823)
- Breeze TTS 2：https://huggingface.co/BreezeBlue/Breeze-TTS-2
- Qwen3-TTS：https://github.com/QwenLM/Qwen3-TTS
- CosyVoice：https://github.com/FunAudioLLM/CosyVoice
- VoxCPM：https://github.com/OpenBMB/VoxCPM ；VoxCPM2 技术报告 [arXiv 2606.06928](https://arxiv.org/abs/2606.06928)；VoxCPM 技术报告 [arXiv 2509.24650](https://arxiv.org/abs/2509.24650)
- IndexTTS：https://github.com/index-tts/index-tts
- OmniVoice：https://github.com/k2-fsa/OmniVoice ；https://huggingface.co/k2-fsa/OmniVoice ；[arXiv 2604.00688](https://arxiv.org/abs/2604.00688)
- Higgs TTS 3：https://huggingface.co/bosonai/higgs-tts-3-4b
- Voxtral TTS：https://huggingface.co/mistralai/Voxtral-4B-TTS-2603 ；[arXiv 2603.25551](https://arxiv.org/abs/2603.25551)
- Step-Audio-EditX：https://huggingface.co/stepfun-ai/Step-Audio-EditX
- Chatterbox：https://github.com/resemble-ai/chatterbox ；https://huggingface.co/ResembleAI/chatterbox-turbo
- VibeVoice：https://github.com/microsoft/VibeVoice
- Pocket TTS：https://github.com/kyutai-labs/pocket-tts ；技术报告 / 博客：https://kyutai.org/blog/2026-01-13-pocket-tts
- Supertonic 3：https://huggingface.co/Supertone/supertonic-3
- Magpie TTS：https://huggingface.co/nvidia/magpie_tts_multilingual_357m
- Kokoro-82M：https://huggingface.co/hexgrad/Kokoro-82M ；音色列表：https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md
- Raon-OpenTTS：https://github.com/krafton-ai/RAON-OpenTTS ；[arXiv 2605.20830](https://arxiv.org/abs/2605.20830)
- Hugging Face Hub API（仓库创建时间、许可证标签）：https://huggingface.co/api/models/<repo>

**X（交叉印证）**
- AA：Eleven v4 Turbo 登顶 https://x.com/ArtificialAnlys/status/2107267465604649164
- AA：HeyGen Voice 登顶 Controlled 榜 https://x.com/ArtificialAnlys/status/2108608255387935199 ；发音鲁棒性 https://x.com/ArtificialAnlys/status/2108608264317599814
- AA：Drama 3 Preview 成为开放权重第 1 https://x.com/ArtificialAnlys/status/2110081938140582085
- Pocket TTS 训练代码开源讨论：https://x.com/daivbr/status/2092466396143722501

> 说明：“HF 仓库创建于”指 Hugging Face API 返回的 createdAt，可能早于正式公开发布的日期；正式发布日期以 README、博客中的 News 条目为准。
