# pstack Skill 深度调研报告

> **主出处**：[`backnotprop/pstack`](https://github.com/backnotprop/pstack)（镜像自 [`cursor/plugins/pstack`](https://github.com/cursor/plugins/tree/main/pstack)），commit `02b59d7`，`plugin.json` **v0.15.32**，MIT。抓取：北京时间（UTC+8）2026-10-26。
>
> **读法**：正文讲 **skill 本身**（概念、工作流、怎么用、原文对照）。**分端安装/用法 → 附录 A**；**作者/社区二级资料 → 附录 B**；完整 URL → 附录 C。
>
> **标注**：【未在原文找到】= 仓库与已抓二级文均未确认；【二级资料】= 博客/作者文，计数可能落后于仓库（以 clone 实测为准）。拼写：**poteto**（非 potato）。

---

## 1. 极简摘要

pstack 是一套 Agent Skills（`skills/<name>/SKILL.md`）：把理解 → 设计 → 实现 → 验证 → 审查 → 交付编成可路由的 **playbook** 与 **principle**，目标是**少写、更高质量、可验证**，从而敢并行多 agent。

| 你要做的 | 命令 |
|---|---|
| 配模型/预算（一次） | `/setup-pstack` |
| 严谨任务默认入口 | `/poteto-mode <目标>. <可判定完成条件>` |
| 不知道用哪个 skill | `/poteto-help <问题>`（只给提示词，不启动重活） |

启用 skill 的最短路径：Cursor `/add-plugin pstack`，或跨端 `npx skills add backnotprop/pstack`；其余宿主与 Grok Bot 细节见 **附录 A**。

仓库实测规模：**58** 个 skill、**24** 条 `principle-*`、**23** 个 playbook、2 个子代理（`poteto-agent`、`comment-sicko`）。

---

## 2. 是什么、解决什么（短）

作者 **poteto**（Lauren Tan，[@poteto](https://x.com/poteto)）：AI 易产 **slop**；pstack 是她在 Cursor 日常用的同一套 skills——「写更少但更好」，并在可验证后做 **fearless parallelism**。默认按模型强弱分工（代码类常走 Grok，判断/文案常走 Opus；`/setup-pstack` 可改）。

| 常见失败 | skill / 原则应对 |
|---|---|
| 未理解就改 | `/how`、`/why`、Investigation |
| 「修好了」无证据 | Prove It Works；`/create-verification-skill` |
| 并行踩同一 worktree | Separate Before Serializing Shared State |
| 长任务不可审计 | `/show-me-your-work` |
| 反复同一错 | `/correct` |
| AI 腔文案 | `/unslop`（mode 对回复也强制） |

叙事背景（验证=限制步骤、overnight 等）见 **附录 B**，正文不展开。

---

## 3. 核心概念与工作流（短而硬）

### 3.1 术语

| 术语 | 含义 |
|---|---|
| **pstack** | 技能包总称 |
| **`/poteto-mode`** | 路由器：匹配 playbook → 步骤进 todo → 按需调其它 skill |
| **playbook** | 一类任务的逐步剧本（23 个 md） |
| **principle-\*** | 24 条短原则；mode 内联索引，用时读 leaf |
| **`/setup-pstack`** | 写 per-role 模型规则 |
| **`/poteto-help`** | 答疑 + 可发送提示词 |
| **poteto-agent** | playbook 步骤里的委托子代理（须先读完 poteto-mode） |
| **arena / swarm** | 同题竞标嫁接 / 切片并行出一份报告 |
| **`/loop`** | Cursor/Claude Code **内置**，非 pstack skill |
| **Benny** | 休眠 Automations 包；**Cursor-only** |

### 3.2 主流程

```mermaid
flowchart TD
  A[目标 + 可判定完成条件] --> B[/setup-pstack]
  B --> C[/poteto-mode]
  C --> D[读 Principles 索引]
  D --> E[匹配 playbook]
  E --> F[步骤原样写入 todo]
  F --> G[路由 how / architect / arena / ...]
  G --> H[poteto-agent + 分角色模型]
  H --> I[真实产物验证]
  I --> J[Opening a PR / 报告]
```

Guide 十章（`docs/guide/`）是叙事教程；日常不必按章点菜——多数时候只开 `/poteto-mode`。

### 3.3 默认模型角色（规则形状节选）

出处：`skills/setup-pstack/SKILL.md`。

```text
# budget: large (xhigh)
feature, refactoring: grok-4.7-xhigh-fast
bug-fix / perf-issue / hillclimb: grok-4.7-xhigh-fast
judgment and prose / hardest tasks: claude-opus-5-5-xhigh
arena runners / interrogate reviewers: opus + grok
swarm workers: grok-4.7-xhigh-fast
```

`inherit-parent` / `auto` = 继承父会话模型（**不是**模型 slug）。配置路径因宿主而异 → **附录 A**。

---

## 4. 怎么用 skill（主戏）

### 4.1 三个入口

**`/setup-pstack`**  
检测本会话可派给子代理的模型 → 选 budget（unlimited / large / medium / small）→ 确认各 role → 写入规则文件。可附带一次要约：生成 `/create-verification-skill`、`/create-formal-verification-skill`。

**`/poteto-mode`**（`disable-model-invocation: true`，须用户主动唤起）

1. 读内联 Principles 索引；  
2. 匹配 playbook，**步骤原文**进 todo（跳过则留 `skip: <reason>`）；  
3. 按步路由其它 skill；代码委托 `poteto-agent`；  
4. 验证须对真实产物；改代码类 playbook 收尾走 Opening a PR。

好提示词：目标 + 完成判定 + 已知线索 + 约束（如 `repro first`、`don't change any code yet`）。**不要**手写「先 /how 再 /architect 再 /arena」——指南 pitfalls：易打乱 playbook 顺序。

**`/poteto-help`**  
搞清场景后给答案 + 一条你可粘贴发送的提示 + 源文件链接；**不替你开跑**（省 token）。

### 4.2 23 个 playbook（做什么用）

完整 md：[`skills/poteto-mode/playbooks/`](https://github.com/backnotprop/pstack/tree/main/skills/poteto-mode/playbooks)

| 组 | playbook |
|---|---|
| 理解 | investigation |
| 构建 | bug-fix, perf-issue, hillclimb, feature, refactoring, prototype, visual-parity |
| 只诊断 | runtime-forensics, trace-forensics |
| 长程/项目 | autonomous-run, orchestrate, autopilot-full, autopilot-stack, multi-phase-plan |
| 交付 | babysit, shipping, opening-a-pr |
| 会话 | session-pickup, pause-safely, worktree-cleanup |
| 元 | authoring-a-skill, eval |

大迁移/无合适 playbook → **`figure-it-out`**（定制可审计剧本）；真正多日多 PR 舰队 → **Orchestrate**（勿把单会话任务误升）。

### 4.3 原则怎么用（24）

不背命令。mode 启动读索引；中途用原则名转向，例如：

```text
apply prove it works. show me the real command output, not the build log.
```

四组（README）：**core**（laziness、foundational-thinking、subtract-before-you-add…）· **architecture**（model-the-domain、boundary-discipline、type-system-discipline…）· **verification**（prove-it-works、fix-root-causes、explain-the-number…）· **delegation/meta**（guard-the-context-window、never-block-on-the-human、encode-lessons-in-structure）。Leaf：`skills/principle-<kebab>/SKILL.md`。

### 4.4 情境 skill 速查 + 原文对照

锚点基址：`https://github.com/backnotprop/pstack/blob/main/`

| Skill | 何时直接用 | 路径 |
|---|---|---|
| how / why / teach / recall | 机制 / 历史证据 / 讲解 / 重建上下文 | `skills/how` 等 |
| architect | 跨函数边界前定类型与模块形 | `skills/architect` |
| arena | 同题 N 候选 → base + 嫁接 | `skills/arena` |
| swarm | 切片或竞速 → 一份 PASS/ISSUES/BLOCKED 报告 | `skills/swarm` |
| interrogate | 多模型对抗审 diff；分 act-on / dismissed | `skills/interrogate` |
| tdd | 便宜本地测试路径或显式要求 TDD | `skills/tdd` |
| benchmark-checklist | 报速度数字前体检 | `skills/benchmark-checklist` |
| blast-radius | 小改动外溢风险，用跑代码证明「为何安全」 | `skills/blast-radius` |
| create- / maintain-verification-skill | 项目级「像用户开 app」证明 | 同名目录 |
| create- / maintain-formal-verification-skill + statechart | 测试难及的性质 / 状态图 | 同名 |
| unslop / bro / technical-writing / no-comments | 去 AI 腔、白话复述、文档标准、Comment Sicko | 同名 |
| figure-it-out / show-me-your-work | 定制剧本 / 决策 TSV | 同名 |
| correct / automate-me / reflect / sharpen | 结构性纠错 / 个人 -mode / 改 skill / 加深提示 | 同名 |
| ketchup / ketchup-video | 追进度（视频版） | 同名 |
| make-bot-ui | 页面按钮经本机服务器 webhook 唤醒 **Grok Bot** | `skills/make-bot-ui` |
| typescript-best-practices | TS 落地 type-system-discipline | 同名 |

**必读原文文件**

| 文件 | 链接 |
|---|---|
| README | [README.md](https://github.com/backnotprop/pstack/blob/main/README.md) |
| poteto-mode | [skills/poteto-mode/SKILL.md](https://github.com/backnotprop/pstack/blob/main/skills/poteto-mode/SKILL.md) |
| setup-pstack | [skills/setup-pstack/SKILL.md](https://github.com/backnotprop/pstack/blob/main/skills/setup-pstack/SKILL.md) |
| poteto-help | [skills/poteto-help/SKILL.md](https://github.com/backnotprop/pstack/blob/main/skills/poteto-help/SKILL.md) |
| Guide 索引 | [docs/guide/README.md](https://github.com/backnotprop/pstack/blob/main/docs/guide/README.md) |
| Claude 适配 | [docs/harness/claude-code.md](https://github.com/backnotprop/pstack/blob/main/docs/harness/claude-code.md) |

**poteto-mode frontmatter 摘录**

```yaml
name: poteto-mode
description: poteto's agent style for concise, detailed responses, deliberate
  subagents, unslopped prose, simple code, and verified work.
disable-model-invocation: true
```

**镜像可独立装的 skill**（不互相硬依赖）：`unslop`, `bro`, `how`, `tdd`, `typescript-best-practices`, `arena`, `swarm`, `interrogate`, `reflect`, `show-me-your-work`, `figure-it-out`, `automate-me`, `correct`, `benchmark-checklist`, `in-your-own-words`, `sharpen`, `statechart`。  
**`poteto-mode`** 需几乎全部 `principle-*` 与多数其它 skill 同装。

### 4.5 示例提示（仓库/guide）

```text
/poteto-mode scroll drifts every 750ms when idle. repro first, then fix and verify.

/poteto-mode add a --json flag. text output stays byte-identical. verify both.

/how do we cancel runs?
/swarm check every package under packages/ against its check.sh. one worker per package.
/interrogate review this pr. no nitpicks unless bug or regression.
/poteto-help which skill should i use to review this branch?
```

### 4.6 仓库明示「不在此包」

`deslop` / `control-cli` / `control-ui` → 另装 `cursor-team-kit`；`/create-skill`、`/loop`、Cursor 内置 babysit → 平台能力。无默认「规划 skill」——作者更信代码与验证；需要时由 mode 覆盖。

---

## 5. 注意事项与边界（短）

- **小改动**不必上全套 mode（token/时间成本）。  
- **完成条件**须可判定；忌「make it better」开 `/loop`。  
- **并行写**：分 worktree / 输出目录；勿共写一棵树。  
- **arena ≠ swarm**：同题竞标 vs 覆盖切片。  
- **绿构建 ≠ 行为证明**。  
- **`auto` 不是模型名**。  
- 非 Cursor：Automations / Bugbot 配置 / 部分 cloud 步骤会降级或 skip（见附录 A）。  
- Grok Bot：**镜像 Install 列表未逐步写 Grok Bot 安装**【未在原文找到】；作者导读称可用；`/make-bot-ui` 在仓库中（附录 A.5 / B）。  
- Flavio 等文的 skill/playbook **计数可能偏旧**；以本报告仓库实测为准。

---

## 6. 参考来源（精简主链）

1. <https://github.com/backnotprop/pstack>  
2. <https://github.com/cursor/plugins/tree/main/pstack>  
3. <https://github.com/backnotprop/pstack/blob/main/README.md>  
4. <https://github.com/backnotprop/pstack/blob/main/skills/poteto-mode/SKILL.md>  
5. <https://github.com/backnotprop/pstack/blob/main/skills/setup-pstack/SKILL.md>  
6. <https://github.com/backnotprop/pstack/blob/main/docs/guide/README.md>  
7. <https://agentskills.io> · <https://skills.sh>  

完整 URL、二级文、空白项 → **附录 B / C**。

---

# 附录 A · 分端安装与用法

### A.0 最短启用

| 方式 | 命令 |
|---|---|
| Cursor | `/add-plugin pstack` → `/setup-pstack` → **新 chat** → `/poteto-mode …` |
| 跨端 CLI | `npx skills add backnotprop/pstack`（自选 skill 与 agents） |
| Claude Code（官方 marketplace） | `/plugin marketplace add cursor/plugins` → `/plugin install pstack@cursor-plugins`；命令带 `pstack:` 前缀 |
| 模型规则路径 | Cursor：`~/.cursor/rules/pstack-models.mdc`；Claude Code：`~/.claude/rules/pstack-models.md`；Codex/Pi/OpenCode 等：`~/.agents/pstack-models.md` |

Marketplace：<https://cursor.com/marketplace/cursor/pstack>

### A.1 Cursor

- Custom Mode：选 `/poteto-mode` 后 **Option+Enter / Alt+Enter**（普通 Enter 只挂一条消息）。  
- 可与内置 `/loop` 配合做 overnight（`/loop` 非 pstack skill）。  
- 子代理：`subagent_type: "poteto-agent"`；勿用 `generalPurpose` 替代；勿派 poteto-agent 跑整个 playbook。  
- 可选同装 `cursor-team-kit`。

### A.2 Claude Code

- 适配表：[`docs/harness/claude-code.md`](https://github.com/backnotprop/pstack/blob/main/docs/harness/claude-code.md)（`Task`→`Agent` 等）。  
- Cursor-only 步骤输出一行 `Skipped in Claude Code: …` 后继续。  
- 第三方 marketplace 默认可关自动更新。  
- 【二级资料】社区 port：`michael-denyer/pstack-claude`（`/plugin marketplace add michael-denyer/pstack-claude`）。

### A.3 Codex

- 镜像声明支持；`spawn_agent`；skills 常在 `~/.codex/skills/` 或 `~/.agents/skills/`。  
- transcript：`~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`（按 `payload.cwd` 过滤）。  
- 【二级资料】port：`codex plugin marketplace add michael-denyer/pstack-claude` → `codex plugin add pstack@pstack-claude`；hook 需 `/hooks` 信任。

### A.4 Pi / OpenCode

- 镜像 `npx skills` 列表含二者。  
- setup 可参考 `pi --list-models`、`opencode models`；OpenCode 提问工具名 `question`，子代理 `task`。  
- Pi 无 subagent 扩展时：委托步骤串行自做（poteto-mode Harness）。  
- 【二级资料】Pi：`pi install git:github.com/michael-denyer/pstack-claude`。

### A.5 Grok Bot

| 点 | 结论 |
|---|---|
| `npx skills` 官方列表含 Grok Bot | 【未在原文找到】 |
| README/guide 安装章逐步装 Grok Bot | 【未在原文找到】 |
| 作者《Complete Guide Pt.1》 | 【二级资料】写「在 Grok Bot 里也能用 pstack」，示例为派 cloud agent + `/poteto-mode` + 验证 skill |
| 仓库 skill | `/make-bot-ui`：本机服务器持 webhook URL+key，浏览器不碰 key；可选 Tailscale |
| 本机会话 skill 目录 | 可能已有同名 poteto-mode 等（交叉旁证），**不能代替仓库安装文档** |

### A.6 其它

仓库未点名的宿主：**写「仓库未提及」**，勿臆造命令。Benny 仅 Cursor Automations。镜像相对上游的社区修复见 README「What this mirror changes」（#14 `#15`）。

---

# 附录 B · 二级资料（Guide / poteto·potato / 社区）

正文只认仓库；下列供对照，**计数以仓库为准**。

| 标题 | 来源 | URL | 状态 |
|---|---|---|---|
| A deep dive into pstack | Flavio Copes · 2026-09-29 | https://flaviocopes.com/pstack/ | 已取全文；计数偏旧 |
| What is poteto-mode | Flavio Copes · 2026-10-09 | https://flaviocopes.com/poteto-mode/ | 已取全文 |
| Loops You Can Trust | poteto · 2026-06-24 | https://pstack.ganhai.cloud/from-poteto/loops-you-can-trust-zh/ | 已取中译；验证=限制步骤 |
| Complete Guide to pstack Pt. 1 | poteto · 2026-09-01 | https://pstack.ganhai.cloud/from-poteto/complete-guide-pt-1-zh/ | 已取中译；验证 skill、Grok Bot、Feature Map |
| Complete Guide Pt. 2 | poteto | https://pstack.ganhai.cloud/from-poteto/complete-guide-pt-2-zh/ | 【未取到全文】 |
| How I Use Cursor | poteto | 收录于 https://pstack.ganhai.cloud/en/from-poteto/ | 【未取到全文】 |
| learn-pstack FAQ | 非官方中文站 | https://pstack.ganhai.cloud/en/faq/ | **potato ≠ poteto** |
| 认识 pstack | learn-pstack | https://pstack.ganhai.cloud/en/ | 已取 |
| DeepWiki Overview | 自动 wiki | https://deepwiki.com/backnotprop/pstack | 已取；Installation 页 fetch 超时【未取到全文】 |
| Cursor HQ 短文 | 2026-10-22 | https://cursorhq.ai/guides/make-cursor-agents-reproduce-fix-and-verify-a-bug-before-they-say-they-r-bd229f/ | 已取；误写 **potato mode** |
| pstack-claude | Michael Denyer | https://github.com/michael-denyer/pstack-claude | README 已取 |
| LinkedIn 开源帖 | Lauren Tan | https://www.linkedin.com/posts/laurenelizabethtan_cursor-pstack-the-skills-behind-10000-activity-7465439429042737152-ADbv | 摘要级 |
| 合入 PR | cursor/plugins#73 | https://github.com/cursor/plugins/pull/73 | 已取描述 |

**作者数字**（如每月约 2000 PR）：仅自述，非独立评测。

---

# 附录 C · 空白项、行动清单、完整 URL

### C.1 【未在原文找到】/【未取到全文】汇总

- 镜像 Install 逐步说明 **Grok Bot** 插件安装；`npx skills` 列表含 Grok Bot  
- 官方命令名 **potato-mode**（仅有 poteto；二级文偶发误写）  
- 《How I Use Cursor》《Complete Guide Pt. 2》全文；DeepWiki Installation 整页  
- `gh release` 资产列表（版本以 `plugin.json` 0.15.32 为准）

### C.2 最小行动清单

1. 装插件或 `npx skills add backnotprop/pstack`（附录 A）  
2. `/setup-pstack` → 新会话  
3. 小真实任务：`/poteto-mode … verify …`  
4. 宜接受 `/create-verification-skill`  
5. 卡住：`/poteto-help`  
6. 对照正文 §4 链接；叙事再读附录 B

### C.3 完整 URL 表

**仓库**  
https://github.com/backnotprop/pstack · https://github.com/cursor/plugins/tree/main/pstack · https://github.com/backnotprop/pstack/blob/main/README.md · https://github.com/backnotprop/pstack/blob/main/MIRROR.md · https://github.com/backnotprop/pstack/blob/main/docs/guide/README.md · https://github.com/backnotprop/pstack/blob/main/docs/guide/01-setup.md · https://github.com/backnotprop/pstack/blob/main/docs/guide/02-poteto-mode.md · https://github.com/backnotprop/pstack/blob/main/docs/guide/07-overnight.md · https://github.com/backnotprop/pstack/blob/main/docs/guide/10-recipes-and-pitfalls.md · https://github.com/backnotprop/pstack/blob/main/docs/harness/claude-code.md · https://github.com/backnotprop/pstack/blob/main/skills/poteto-mode/SKILL.md · https://github.com/backnotprop/pstack/blob/main/skills/setup-pstack/SKILL.md · https://github.com/backnotprop/pstack/blob/main/skills/poteto-help/SKILL.md · https://github.com/backnotprop/pstack/blob/main/skills/make-bot-ui/SKILL.md · https://github.com/backnotprop/pstack/blob/main/plugin.json · https://agentskills.io · https://skills.sh · https://cursor.com/marketplace/cursor/pstack · https://cursor.com/docs/skills#using-a-skill-as-a-custom-mode · https://github.com/cursor/plugins/pull/73  

**二级**  
https://flaviocopes.com/pstack/ · https://flaviocopes.com/poteto-mode/ · https://flaviocopes.com/unslop/ · https://pstack.ganhai.cloud/en/ · https://pstack.ganhai.cloud/en/faq/ · https://pstack.ganhai.cloud/en/from-poteto/ · https://pstack.ganhai.cloud/from-poteto/loops-you-can-trust-zh/ · https://pstack.ganhai.cloud/from-poteto/complete-guide-pt-1-zh/ · https://pstack.ganhai.cloud/from-poteto/complete-guide-pt-2-zh/ · https://www.linkedin.com/posts/laurenelizabethtan_cursor-pstack-the-skills-behind-10000-activity-7465439429042737152-ADbv · https://deepwiki.com/backnotprop/pstack · https://deepwiki.com/backnotprop/pstack/1.1-installation-and-setup · https://cursorhq.ai/guides/make-cursor-agents-reproduce-fix-and-verify-a-bug-before-they-say-they-r-bd229f/ · https://github.com/michael-denyer/pstack-claude · https://x.com/poteto  

---

*改写说明：正文压缩为 skill 主戏；分端用法→附录 A；二级 Guide/Flavio/learn-pstack 等→附录 B；事实未删，仅重排。主出处 `backnotprop/pstack@02b59d7`。*
