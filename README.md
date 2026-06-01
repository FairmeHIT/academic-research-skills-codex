# Academic Research Skills for Codex 简体中文适配版

[![Version](https://img.shields.io/badge/version-v0.1.8-blue)](VERSION)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/license-CC%20BY--NC%204.0-lightgrey)](https://creativecommons.org/licenses/by-nc/4.0/)
[![Language](https://img.shields.io/badge/language-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E9%80%82%E9%85%8D%E7%89%88-red)](README.md)
[![Sponsor](https://img.shields.io/badge/sponsor-Buy%20Me%20a%20Coffee-orange?logo=buy-me-a-coffee)](https://buymeacoffee.com/crucify020v)

`academic-research-skills-codex-zh-CH` 是
[Academic Research Skills for Claude Code](https://github.com/Imbad0202/academic-research-skills)
面向 Codex 用户的简体中文适配版。它把 ARS 的研究工作流封装为一个 Codex skill：
`$academic-research-suite`。

本适配版默认面向简体中文使用场景：

- 中文输出默认使用简体中文。
- 面向用户的中文文档发布为 `zh-CN`。
- ARS 触发词和路由提示包含简体中文表述。
- CJK LaTeX 指南使用简体中文字体默认配置。

## 项目结构

本仓库将 ARS 工作流内容 vendored 到一个 Codex skill 中：

```text
skills/academic-research-suite/
  SKILL.md
  manifest.json
  agents/openai.yaml
  ars/
    deep-research/
    academic-paper/
    academic-paper-reviewer/
    academic-pipeline/
    experiment-agent/
    commands/
    hooks/
    docs/
    tests/
    shared/
```

原始 Claude Code ARS 仓库不会被直接修改。上游内容来自 GitHub fresh clone，
并通过 `skills/academic-research-suite/SKILL.md` 中的 Codex router 进行适配。

## 与 Claude Code 版本的关系

本仓库是 Codex 发行版。原始 Claude Code 版本请使用：
[Imbad0202/academic-research-skills](https://github.com/Imbad0202/academic-research-skills)。

如果你需要 Claude Code 原生 skill 布局、Claude 专用 agent-team 行为，或完整的
ARS 上游开发历史，请使用原始仓库。如果你需要 Codex 原生的单 skill 入口，请使用本仓库。

## 版本说明

当前 Codex package 版本为 `0.1.8`。以下位置共同记录 Codex package 版本：

- 根目录 `VERSION`
- `skills/academic-research-suite/SKILL.md` metadata version
- `skills/academic-research-suite/manifest.json` 中的 `adapter_version`

Codex package 版本独立于 vendored ARS suite 版本。上游 vendored 版本通过
`manifest.source_repositories[]` 中的 commit 记录。

包级别变更记录见 [`CHANGELOG.md`](CHANGELOG.md)。

当前 vendored ARS source 跟踪：
`Imbad0202/academic-research-skills@96b82e82142dc95f117595c207d3e150b078e411`
（`v3.9.4.2`）。上游 v3.9.4.2 的增量主要是 `.github/` 下的 CI/release gate
内容，本 Codex package 有意排除这些文件；运行时内容包含 ARS v3.9.4.1 的
temporal-verification hotfix，以及 v3.9.1 到 v3.9.4 的工作流更新。

## 安装与更新

从本仓库路径安装 skill。建议使用 `--method git`，这样公开 GitHub 访问和带凭据访问
都更稳定。

```bash
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo FairmeHIT/academic-research-skills-codex-zh-CH \
  --ref main \
  --path skills/academic-research-suite \
  --method git
```

更新已有安装：

```bash
rm -rf "$HOME/.codex/skills/academic-research-suite"
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo FairmeHIT/academic-research-skills-codex-zh-CH \
  --ref main \
  --path skills/academic-research-suite \
  --method git
```

安装或更新后，请开启一个新的 Codex conversation。已有 Codex session 可能仍缓存旧版
skill；不需要关闭无关的 Claude 或 Codex session。

使用 `/skills` 验证安装结果：你应该只看到一个 ARS entry，即
`academic-research-suite` 或 `Academic Research ...`。不应看到本 package 分出的
`academic-paper`、`academic-pipeline`、`deep-research` 或
`academic-paper-reviewer` 等独立 skill。如果出现这些条目，请按上面的更新命令重新安装，
并开启新的 Codex conversation。

## 文档

- [Codex 安装与设置](skills/academic-research-suite/ars/docs/SETUP.zh-CN.md)
  说明安装、`ars-*` 别名、可选工具、Material Passport adapters，以及不支持的
  Claude plugin 功能。
- [Codex architecture](skills/academic-research-suite/ars/docs/ARCHITECTURE.md)
  说明 ARS pipeline 的逻辑结构和 Codex runtime overlay。

## 使用方式

显式调用 `$academic-research-suite`，然后描述研究任务，并提供源文件、笔记、草稿、
审稿意见或输出约束。

```text
Use $academic-research-suite to help me plan a systematic literature review on
AI adoption in higher education quality assurance.
```

也可以直接用中文描述任务：

```text
Use $academic-research-suite。

我想写一篇关于高校质量保障中 AI 采用的系统综述。
目前还没有清晰的研究问题，请先通过苏格拉底式提问帮我收敛选题，
不要直接写论文大纲。
```

Codex adapter 会将请求路由到五类 ARS 工作流：

| 工作流 | 适用场景 | 示例 prompt |
|---|---|---|
| `deep-research` | 研究问题收敛、文献综述、系统综述、meta-analysis、事实核查 | `Use $academic-research-suite to build a systematic review protocol for AI in higher education QA.` |
| `academic-paper` | 论文大纲、写作、摘要、修订、引用格式、AI disclosure | `Use $academic-research-suite to turn these notes into an IMRaD paper outline and drafting plan.` |
| `academic-paper-reviewer` | 稿件评审、模拟 peer review、编辑决定、复审 | `Use $academic-research-suite to review this manuscript and produce a journal-style decision letter.` |
| `academic-pipeline` | 从研究到论文的端到端流程，包含完整性检查、评审、修订和最终检查 | `Use $academic-research-suite to run an end-to-end research-to-paper pipeline from topic to revised manuscript.` |
| `experiment-agent` | 代码实验规划、人类研究方案、统计解释、可复现性验证 | `Use $academic-research-suite to plan a code experiment and define reproducibility checks.` |

## Claude 风格别名

Claude Code v3.7 会安装 `/ars-*` slash commands。Codex 没有同样的 plugin command
registry，因此本 package 在单一 `$academic-research-suite` skill 中模拟这些命令意图。

推荐写法：

```text
Use $academic-research-suite: ars-plan my paper on AI governance in universities.
```

如果你的 Codex client 会把 slash-prefixed 文本作为普通用户消息传入，也可以写：

```text
/ars-plan my paper on AI governance in universities.
```

如果 slash input 被 client 拦截，请使用普通 alias：

```text
ars-plan my paper on AI governance in universities.
```

| Claude command | Codex alias | 路由工作流 |
|---|---|---|
| `/ars-plan` | `ars-plan` | `academic-paper` `plan` mode |
| `/ars-outline` | `ars-outline` | `academic-paper` `outline-only` mode |
| `/ars-abstract` | `ars-abstract` | `academic-paper` `abstract-only` mode |
| `/ars-lit-review` | `ars-lit-review` | `academic-paper` `lit-review` mode |
| `/ars-citation-check` | `ars-citation-check` | `academic-paper` `citation-check` mode |
| `/ars-disclosure` | `ars-disclosure` | `academic-paper` `disclosure` mode |
| `/ars-format-convert` | `ars-format-convert` | `academic-paper` `format-convert` mode |
| `/ars-revision-coach` | `ars-revision-coach` | `academic-paper` `revision-coach` mode |
| `/ars-revision` | `ars-revision` | `academic-paper` `revision` mode |
| `/ars-full` | `ars-full` | `academic-pipeline` full workflow |

## 推荐工作方式

为了得到更稳定的输出，请在开头说明工作流目标、材料状态和输出约束：

```text
Use $academic-research-suite。

目标：写一篇期刊论文。
当前材料：已有文献矩阵和粗略发现，但还没有大纲。
当前需要：论文结构和缺失证据清单。
约束：英文写作，APA 7，面向高等教育政策读者。
```

如果只有宽泛主题、还没有清晰研究问题，请先要求 ARS 进行 Socratic scoping：

```text
Use $academic-research-suite。

我想写一篇关于高校质量保障中 AI 采用的论文。
我还没有清晰的研究问题。
请使用 SCR / Socratic dialogue 帮我先收敛问题；暂时不要写大纲。
```

预期路由：先进入 `deep-research` 的 `socratic` mode。ARS 应先提出收敛问题，
不应在研究问题收敛前直接输出大纲或草稿。

评审任务请提供稿件或稿件路径，并说明需要的评审模式：

```text
Use $academic-research-suite to review this paper.
Mode: full review.
Focus: methodology, contribution, citation integrity, and likely desk-reject risks.
Output: reviewer reports plus editorial decision letter.
```

长流程任务建议要求 checkpoint，而不是让 Codex 静默运行完整 pipeline：

```text
Use $academic-research-suite to start an academic-pipeline run.
Begin with Stage 0 intake and stop after producing the pipeline dashboard.
```

## Smoke Tests

在新的 Codex conversation 中运行：

```text
/skills
```

预期结果：只出现一个 ARS entry。

测试 Socratic routing：

```text
Use $academic-research-suite.
I want to write a paper on AI adoption in higher education quality assurance.
I do not yet have a clear research question.
```

预期结果：路由到 `deep-research` 的 `socratic` mode，并提出收敛问题。

CLI smoke test：

```bash
codex exec --ephemeral --sandbox read-only \
  -C /path/to/academic-research-skills-codex-zh-CH \
  'Use $academic-research-suite. Router smoke test only. User request to classify: I want to write a paper on AI adoption in higher education quality assurance, but I do not yet have a clear research question. According to the academic-research-suite router, classify the workflow and mode.'
```

## Codex 常见非阻塞警告

以下 Codex 消息不表示 ARS 安装失败：

- `[features].codex_hooks is deprecated`：可在方便时更新 Codex config；ARS Codex
  正常使用不依赖 hooks。
- `hooks need review before they can run`：如果你需要使用这些 hooks，可单独 review。
  ARS Codex 将 vendored Claude hooks 视为 traceability metadata，不要求它们运行。

## Codex Adapter 行为

ARS 最初为 Claude Code 编写。在本 Codex package 中：

- vendored `agents/*.md` 文件作为角色和阶段 prompt 使用。
- vendored `commands/ars-*.md` 文件只作为 prompt recipes；Codex 不会注册它们为
  slash commands。
- vendored `hooks/hooks.json` 仅用于上游可追溯性；Codex 不会从本 package 安装或执行
  Claude Code hooks。
- Codex 不会自动生成后台 agents，除非你明确要求 delegated 或 parallel agent work。
- 当涉及当前事实或外部事实时，web/source verification 使用 Codex browsing，并需要引用来源。
- Cross-model verification 默认关闭。若在本 Codex package 中明确请求该功能，需要配置
  `ARS_CROSS_MODEL=claude-opus-4.7` 和 `ANTHROPIC_API_KEY`；外部 reviewer 使用
  Anthropic Claude Opus 4.7 API，而不是 Codex/OpenAI API。
- 上游文档中的 “fresh Claude Code session” 在本 package 中表示新的 Codex conversation；
  Material Passport reset semantics 仍然适用。
- 对无法验证的引用、来源、统计数据或期刊政策，Codex 应标记为 unverified，而不是编造依据。

## ARS v3.9.4.2 对齐情况

本 package 尽量在 Codex 有对应概念的地方保持与上游 ARS v3.9.4.2 相同的用户可见工作流内容。

| 上游 ARS 功能 | Codex package 行为 |
|---|---|
| 一个可安装 plugin | 一个可安装 Codex skill：`skills/academic-research-suite` |
| `/ars-*` slash commands | 通过 skill router 模拟为 `ars-*` aliases；不是原生 slash commands |
| 四个上游 skills 由 `skills/` symlinks 自动发现 | 单一 Codex router skill 选择 workflow，并读取 vendored workflow `WORKFLOW.md` |
| Plugin-shipped agents | agent 文件作为 role/phase prompts；除非用户明确要求 subagents，否则 Codex inline 执行 |
| `model: opus` / `model: sonnet` command routing | 视为 Claude metadata；Codex 使用当前 active model |
| SessionStart 和 SubagentStop hooks | 仅为 traceability vendored；Codex 不安装或执行 Claude hooks |
| Plugin marketplace update / auto-update | 不适用于本仓库；通过 reinstall 或 pull 本 Codex repo 更新 |
| Claude Code Agent Team | 不自动启用；Codex subagents 需要用户明确要求 delegation 或 parallel agents |
| 上游 GPT/Gemini cross-model dispatch | 默认关闭；仅支持显式配置后的 Anthropic Claude Opus 4.7 reviewer |

## 可选 Claude Opus 4.7 Reviewer API

如需 reviewer calibration 或 cross-model devil's advocate checks：

```bash
export ANTHROPIC_API_KEY="<your-anthropic-api-key>"
export ARS_CROSS_MODEL="claude-opus-4.7"
```

然后在 prompt 中明确请求 cross-model verification。若缺少任一环境变量，ARS Codex
会回退到单 runtime review，并应报告 Claude Opus 4.7 verifier 不可用。

## 支持与赞助

如果 ARS Codex 对你的研究工作流有帮助，可以通过
[Buy Me a Coffee](https://buymeacoffee.com/crucify020v) 支持维护。

## 安全

请不要在公开 issue 中披露漏洞。私密报告方式见 [`SECURITY.md`](SECURITY.md)，
本地验证摘要见 [release readiness and security report](security_best_practices_report.md)。

## 高级使用的文件布局

入口文件：

```text
skills/academic-research-suite/SKILL.md
```

工作流内容：

```text
skills/academic-research-suite/ars/<workflow>/
```

共享 schema、compliance rules 和 cross-workflow contracts：

```text
skills/academic-research-suite/ars/shared/
```

调试或更新 package 时请保留这些路径。许多 ARS workflow 文件会交叉引用 `shared/`、
`scripts/`、`examples/` 和其他 workflow 目录。

## 更新策略

更新时，将选定的上游 ARS 内容同步到 `skills/academic-research-suite/ars/`。
不要盲目镜像 Claude Code 仓库；应排除 Claude/plugin loader 文件，例如 `.claude/`、
`.claude-plugin/`、`.github/`、源仓库 `.gitignore`，以及 Codex 不需要的 symlink-only
alias directories。

### 未启用的上游脚本

部分上游维护脚本已 vendored，但在本 Codex package 中有意保持 inactive，因为它们依赖
非 vendored 的 Claude Code 输入，例如 `.claude/CLAUDE.md`。将任何上游脚本接入 Codex CI
之前，请先查看 `skills/academic-research-suite/manifest.json` 中的
`inactive_upstream_scripts`。

## 贡献者与致谢

**Cheng-I Wu** - ARS suite 和本 Codex sibling distribution 的维护者。

**Codex** - 在维护者指导下协助完成 Codex adapter packaging、router-policy hardening、
test fixes 和 release-readiness review。

Vendored upstream ARS contributors 见
[`skills/academic-research-suite/ars/README.md`](skills/academic-research-suite/ars/README.md#contributors)。
