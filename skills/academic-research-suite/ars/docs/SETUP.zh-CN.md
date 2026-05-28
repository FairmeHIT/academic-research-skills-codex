# ARS Codex 安装设定

本文说明 `academic-research-skills-codex` 的 Codex 版设定方式，不是
Claude Code plugin 安装指南。Claude Code 原生版本请使用
`Imbad0202/academic-research-skills`。

## 最小可行设定

安装单一 Codex skill。建议使用 `--method git`，让 public 与 credentialed GitHub
存取行为都比较稳定：

```bash
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo Imbad0202/academic-research-skills-codex \
  --ref main \
  --path skills/academic-research-suite \
  --method git
```

更新既有安装：

```bash
rm -rf "$HOME/.codex/skills/academic-research-suite"
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo Imbad0202/academic-research-skills-codex \
  --ref main \
  --path skills/academic-research-suite \
  --method git
```

安装后开新的 Codex conversation。已经开著的 session 可能保留旧的 skill
cache；不用为了更新这个 skill 去关掉其他 Claude 或 Codex session。

使用时明确呼叫：

```text
Use $academic-research-suite to plan a systematic literature review on AI in higher education QA.
```

Skill 名称是单数：`$academic-research-suite`。

用 `/skills` 验证安装。正常状态应该只看到一个 ARS 入口：
`academic-research-suite` 或 `Academic Research ...`。如果看到本 package
额外暴露 `academic-paper`、`academic-pipeline`、`deep-research`、
`academic-paper-reviewer` 等独立 skill，请用上方更新指令重新安装，并开新的
Codex conversation。

一般 Codex 使用不需要 Anthropic API key。主要模型由目前 Codex runtime
提供。`ANTHROPIC_API_KEY` 只用于下方选用的外部 Claude Opus reviewer。

## Claude-style aliases

Claude Code ARS 会安装 `/ars-*` slash commands。Codex 没有同一套 command
registry，所以本 package 在 `$academic-research-suite` router 里模拟同样意图。

如果 Codex client 会拦截 slash input，请用不含 slash 的 alias：

```text
ars-plan my paper on AI governance in universities
ars-revision-coach reviewer_comments.md
ars-full topic: demographic decline and university quality assurance
```

也可以包在明确的 skill 呼叫内：

```text
Use $academic-research-suite: ars-outline for this manuscript draft.
```

| Claude command | Codex alias | 路由 workflow |
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

`commands/ars-*.md` frontmatter 里的 `model: opus` / `model: sonnet` 是
Claude Code routing metadata。Codex 会使用目前 active model，除非使用者在对话中明确指定其他模型。

## 模糊论文题目的 Socratic 收敛

当使用者说想写论文，但只有题目、暂定标题或大方向，还没有明确 research
question 时，Codex router 应比照 upstream ARS：先进 `deep-research`
`socratic` mode，不直接产生大纲、draft 或完整 pipeline。

建议 prompt：

```text
Use $academic-research-suite.
我想做一篇论文，题目方向是 AI adoption in higher education quality assurance。
我还没有明确 research question。
请先用 SCR / Socratic 问答帮我收敛问题，不要先写大纲。
```

预期行为：

- route 到 `deep-research` `socratic` mode
- 先问收敛问题
- 条件足够后才整理 2-3 个候选 RQ
- RQ 清楚前不进入 `academic-paper` 大纲或写作

如果要跑完整 pipeline，但 Stage 1 先 SCR：

```text
Use $academic-research-suite to start academic-pipeline.
Stage 1 请先用 deep-research socratic mode，因为我目前只有模糊主题。
Stop after the RQ Brief and pipeline dashboard.
```

## Smoke tests

Codex app / interactive CLI：

```text
/skills
```

预期：只看到一个 ARS 入口。

Router smoke：

```text
Use $academic-research-suite.
我想做一篇论文，题目方向是 AI adoption in higher education quality assurance。
我还没有明确 research question。
```

预期：进入 `deep-research` `socratic` mode。

Codex CLI：

```bash
codex exec --ephemeral --sandbox read-only \
  -C /path/to/academic-research-skills-codex \
  'Use $academic-research-suite. Router smoke test only. User request to classify: 我想做一篇论文，题目方向是 AI adoption in higher education quality assurance，但我还没有明确 research question。 According to the academic-research-suite router, classify the workflow and mode.'
```

## 不代表安装失败的 Codex 警告

看到下列讯息不代表 ARS 安装失败：

- `[features].codex_hooks is deprecated`：有空再更新 Codex config 即可。ARS
  Codex 正常使用不需要 hooks。
- `hooks need review before they can run`：如果你有使用那些 hooks，再另外审核即可。本
  package vendored 的 Claude hooks 只作 traceability，不会被 Codex 安装或执行。

## 选用本机工具

Markdown 输出不需要额外工具。只有在需要稳定本机转档或 corpus adapter 时才安装下列工具。

### DOCX 输出

直接产生 `.docx` 需要 Pandoc。若没有 Pandoc，ARS Codex 应回退为
Markdown 加转档说明。

```bash
# macOS
brew install pandoc

# Linux (Debian/Ubuntu)
sudo apt-get install pandoc
```

### LaTeX / PDF 输出

PDF 输出需要 `tectonic` 与相关字体。这是选用功能。

```bash
# macOS
brew install tectonic

# Linux (Debian/Ubuntu)
curl --proto '=https' --tlsv1.2 -fsSL https://drop-sh.fullyjustified.net | sh
```

APA 7 中文输出建议字体：

- Times New Roman
- Source Han Serif SC VF / Noto Serif SC
- Courier New

### Adapter 依赖

Material Passport reference adapters 使用 vendored `requirements-dev.txt`：

```bash
cd skills/academic-research-suite/ars
python -m pip install -r requirements-dev.txt
```

## Material Passport `literature_corpus[]` adapters

如果你已有策展文献库，请先在 ARS session 之外跑 adapter，再把产出的
`passport.yaml` 交给 Codex。

从 vendored ARS root 执行：

```bash
cd skills/academic-research-suite/ars

python scripts/adapters/folder_scan.py \
  --input /path/to/pdfs \
  --passport passport.yaml \
  --rejection-log rejection_log.yaml

python scripts/adapters/zotero.py \
  --input my-zotero-export.json \
  --passport passport.yaml \
  --rejection-log rejection_log.yaml

python scripts/adapters/obsidian.py \
  --input ~/Obsidian/Lit\ Notes \
  --passport passport.yaml \
  --rejection-log rejection_log.yaml
```

Consumer protocol 与 upstream ARS 相同：`bibliography_agent` 与
`literature_strategist_agent` 在侦测到非空且可解析的 corpus 时，采
corpus-first / search-fills-gap 行为。详见
[`academic-pipeline/references/literature_corpus_consumers.md`](../academic-pipeline/references/literature_corpus_consumers.md)。

## 选用环境变数

所有 flag 都是 opt-in。

| Flag | Codex 行为 |
|---|---|
| `ARS_SOCRATIC_READING_PROBE=1` | workflow 进入 `socratic_mentor_agent` prompt 时，启用 Socratic reading-check probe。 |
| `ARS_PASSPORT_RESET=1` | 将 FULL checkpoint 提升为 Material Passport reset boundary。Upstream 的「fresh Claude Code session」在这里代表新的 Codex conversation。 |
| `ARS_CLAIM_AUDIT=1` | workflow 进入 upstream 对应路径时，启用选用的 v3.8 claim-reference alignment audit gate。 |
| `ARS_CROSS_MODEL=claude-opus-4.7` + `ANTHROPIC_API_KEY` | 使用者明确要求时，启用选用的外部 Claude Opus reviewer。 |
| `ARS_CROSS_MODEL_SAMPLE_INTERVAL` | 明确启用 cross-model review 时的 advisory sampling interval。 |
| `S2_API_KEY` | 选用 Semantic Scholar key，用于加速 reference lookup 与 contamination-signal migration。 |
| `OPENALEX_POLITE_EMAIL`, `CROSSREF_POLITE_EMAIL` | ARS v3.9.0 OpenAlex / Crossref triangulation clients 的选用 polite-pool 识别资讯。 |

Upstream GPT/Gemini cross-model 范例在 Codex package 里不启用。不要为
ARS Codex cross-model review 设定 `OPENAI_API_KEY` 或 `GOOGLE_AI_API_KEY`；
Codex 本身已提供主要 OpenAI model。

## 无法直接复制的 Claude Code 功能

| Claude Code 功能 | Codex 状态 |
|---|---|
| `/plugin marketplace add` / `/plugin install` | 不支援。请把本 repo 安装为 Codex skill。 |
| 原生 `/ars-*` slash command 注册 | 不支援。改用 `$academic-research-suite` 内的 `ars-*` alias。 |
| `skills/` symlink 自动探索 | 改为单一 Codex router skill。 |
| plugin-shipped `agents/` | 作为 role prompt 内联使用，不自动 dispatch。 |
| Claude Code Agent Team / Task tool | Codex subagents 只有在使用者明确要求委派或平行 agent work 时才使用。 |
| SessionStart / SubagentStop hooks | 仅保留作 upstream traceability，不在 Codex 安装或执行。 |
| `model: opus` / `model: sonnet` command routing | 不作 routing 控制；Codex 使用目前 active model。 |
| plugin auto-update | 不支援；更新方式是重新安装或 pull Codex repo。 |

## 开发验证

从 Codex repo root 执行：

```bash
python -m json.tool skills/academic-research-suite/manifest.json
python skills/academic-research-suite/ars/scripts/check_data_access_level.py --path skills/academic-research-suite/ars
python skills/academic-research-suite/ars/scripts/check_task_type.py --path skills/academic-research-suite/ars
```

Upstream pytest 请从 vendored ARS root 执行，因为测试会 import
`scripts.*`：

```bash
cd skills/academic-research-suite/ars
python -m pytest --tb=short -q
```

Codex package 已在可行处补 nested path lookup。第一次 vendor、但本 repo Git
history 尚未包含 upstream v3.6.7 manifest 时，
`scripts/check_v3_6_8_pattern_protection.py` 会 fallback 到
`scripts/codex_v3_6_7_block_baseline.json`，因此 byte-equivalence check 仍会抓到
protected block mutation，不会因为缺 Git baseline 而自我基准化。
