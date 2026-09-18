# Session Handoff

> A portable Agent Skill and plugin for safe handoff and recovery across ChatGPT, Codex, and other compatible agent runtimes.

## 中文

### 它解决什么问题

长对话结束时，普通摘要通常只回答“之前做了什么”，却没有回答：当前项目真实状态是什么、交接里的结论是否仍然有效、机器验证和人工确认是否分别完成，以及下一步是否真的安全。

`session-handoff` 把会话交接定义为一个需要核验的状态边界：

> handoff 是 anchor（锚点），不是 absolute fact（绝对事实）。

下一位 Agent 必须读取真实项目资产，比较当前状态与交接锚点，并在证据不足或发生冲突时停止，而不是直接沿用旧摘要继续执行。

### 核心特点

| 能力 | 行为 |
|---|---|
| 双模式 | `prepare` 负责生成交接；`recover` 负责接手前核验 |
| 证据地图 | 每个关键结论都关联到文件、版本、验证记录或明确的缺失证据 |
| 双重门禁 | 分离 `Machine Gate` 与 `Human Confirmation Gate`；自动化通过不等于人工批准 |
| 三态判定 | `MATCH`、`CONFLICT`、`INSUFFICIENT_EVIDENCE`，并规定冲突优先级 |
| 安全停止 | `recover` 首轮只读、生成 `Recovery Verification Report`，然后等待明确确认 |
| 持久化 | 可写且获准时写入 `.handoff/LATEST.md` 与 `.handoff/history/`；否则降级为 `chat-only` |
| 最小依赖 | V1 只有 Skill 和 references，不需要 MCP、守护进程、后台监控或 hook |

### V1.0.1 修订

`V1.0.1` 强化了 recover 报告的结构契约：逐项核验可以同时出现 `MATCH`、`CONFLICT` 和 `UNKNOWN`，但整份报告只能发布一个总体 verdict。总体判定固定按 `CONFLICT` → `INSUFFICIENT_EVIDENCE` → `MATCH` 的顺序计算，并在输出前核对双重门禁、Evidence Map、单一 Next Action、Do Not Do 和最终 Human Confirmation Gate 是否齐全。

### 两种模式

#### `prepare`：离开前建立交接锚点

适合“我要切换会话”“请保存当前断点”“帮下一位 Agent 接手”。它会记录：

- 当前目标、完成情况和阻塞项；
- 当前 revision、工作区、文件和验证记录；
- `Machine Gate` 与 `Human Confirmation Gate`；
- Evidence Map、已确定的决策、开放问题；
- 一个且只有一个安全的 `Next Action`；
- 明确的 `Do Not Do`；
- 持久化位置，或无法写入时的 `chat-only` 原因。

#### `recover`：接手前核验真实状态

适合“从上次交接恢复”“继续之前的任务，但先核对当前项目”。它会：

1. 读取 handoff、适用的项目规则和当前项目资产；
2. 对比交接锚点与真实状态；
3. 输出 `Recovery Verification Report`；
4. 给出且仅给出一个总 verdict：`MATCH`、`CONFLICT` 或 `INSUFFICIENT_EVIDENCE`；
5. 保留 Evidence Map、Next Action、Do Not Do 和人工确认门；
6. 停止，等待用户明确确认后才继续。

`CONFLICT` 优先于 `INSUFFICIENT_EVIDENCE`：只要当前可靠证据直接反驳任一关键交接结论，总 verdict 就是 `CONFLICT`。没有证据但没有直接反驳时，才是 `INSUFFICIENT_EVIDENCE`。

### 使用方式

在网页版 ChatGPT 安装插件后：

```text
@session-handoff prepare
@session-handoff recover
```

在 Codex 中：

```text
$session-handoff prepare
$session-handoff recover
```

也支持中文自然语言，例如：

```text
我要切换会话，请为当前项目准备交接。
请从 .handoff/LATEST.md 恢复，先核对真实状态，不要直接继续执行。
```

### 安装与分发

插件目录采用可移植结构：

```text
plugins/session-handoff/
├── plugin.json
├── .codex-plugin/plugin.json
└── skills/session-handoff/
    ├── SKILL.md
    └── references/
        ├── handoff-template.md
        └── recovery-template.md
```

网页版 Chat/Work 不会直接读取某台电脑上的 `~/.codex/skills`。要在工作区使用：

1. 将本仓库导入工作区的插件市场；
2. 在 ChatGPT 的“插件”目录安装 `Session Handoff`；
3. 新建聊天后，用 `@session-handoff` 显式调用。

本仓库提供 `.agents/plugins/marketplace.json`，可供工作区管理员导入；本地 Codex 也可以直接从 `plugins/session-handoff` 测试。

### 与其他 handoff 类 Skill 的区别

下表基于公开项目 README 和 SKILL.md 的可见行为进行比较：

| 项目 | 主要设计 | 与本项目的区别 |
|---|---|---|
| [context-handoff](https://github.com/djhyes/context-handoff) | 通过 `PreCompact` / `Stop` hook 自动提醒和生成 handoff，并提供跨运行时适配 | 本项目不依赖 hook 或守护机制；重点是 recover 时的实时状态核验、门禁分离和人工确认 |
| [codex-session-handoff](https://github.com/lifang336/codex-session-handoff) | 在 `.codex/handoffs/` 中保存项目交接，并提供 create/resume 流程 | 本项目使用 `.handoff/LATEST.md + history`，并把 handoff 明确视为待验证锚点，而不是直接可信的项目事实 |
| [codex-session-handoff-skill](https://github.com/Liu-Bot24/codex-session-handoff-skill) | 创建本地快照、索引和下一会话提示词，覆盖代码与非代码任务 | 本项目不做 transcript/session 文件抽取；它要求下一 Agent 对当前项目资产做 evidence-backed comparison，并在首轮停止 |
| [handoff](https://github.com/lhh666-6/codex-skills/tree/main/skills/handoff) | 写入一次性的 `handoff.md`，由 SessionStart hook 读取并删除 | 本项目默认持久保留 `LATEST + history`，不自动删除，不把一次读取视为完成核验 |
| [continuity-handoff](https://github.com/ciumbar/continuity-handoff) | 面向 Codex、Claude、OpenClaw 等运行时的 provider-neutral 连续性文件 | 本项目优先解决“能否安全继续”的证据和权限问题，而不是提供跨平台适配层 |

因此，它不是“更长的摘要模板”，也不是“自动把旧会话全文搬过去”。它的差异集中在三个约束：

1. 以真实项目状态校验交接锚点；
2. 将机器验证与人工确认拆成两个独立门禁；
3. 在 `MATCH`、`CONFLICT`、`INSUFFICIENT_EVIDENCE` 后明确停住，等待人类确认。

### 非目标（V1）

- 不自动读取或重放完整聊天 transcript；
- 不替用户解决状态冲突；
- 不绕过权限、审批、发布或外部消息授权；
- 不提供 watcher、daemon、自动监控或后台继续执行；
- 不把旧测试结果、旧 commit 或“看起来像批准”的文字当作当前事实。

### 开发与验证

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py \
  plugins/session-handoff/skills/session-handoff

python3 /path/to/plugin-creator/scripts/validate_plugin.py \
  plugins/session-handoff
```

验证重点不是 README 的关键词，而是可观察行为：prepare 是否形成完整交接，recover 是否读取真实资产、正确分类并在报告后停止。

## English

### What problem it solves

When a long agent conversation ends, a normal summary may say what happened without proving what is true now. The successor still needs to know whether the handoff is current, whether automated checks and human approval are separate, and whether the next action is safe.

`session-handoff` treats a handoff as a state boundary that must be verified:

> A handoff is an anchor, not an absolute fact.

The receiving agent reads the live project assets, compares them with the handoff anchor, and stops when evidence is missing or contradictory instead of blindly continuing from an old summary.

### Core capabilities

| Capability | Behavior |
|---|---|
| Two modes | `prepare` creates a handoff; `recover` verifies it before continuation |
| Evidence Map | Decision-critical claims point to files, revisions, records, or explicit missing evidence |
| Two gates | `Machine Gate` and `Human Confirmation Gate` stay independent |
| Three verdicts | `MATCH`, `CONFLICT`, and `INSUFFICIENT_EVIDENCE` with explicit precedence |
| Safe stop | The first `recover` pass is read-only, emits a `Recovery Verification Report`, then waits |
| Persistence | Writes `.handoff/LATEST.md` and `.handoff/history/` when safely permitted, otherwise reports `chat-only` |
| Minimal V1 | Instruction-only; no MCP server, daemon, watcher, background monitor, or hook |

### V1.0.1 revision

`V1.0.1` strengthens the recover report contract. Row-level findings may mix `MATCH`, `CONFLICT`, and `UNKNOWN`, while the report publishes exactly one overall verdict. The overall result is computed in `CONFLICT` → `INSUFFICIENT_EVIDENCE` → `MATCH` precedence, followed by a structural check for both gates, the Evidence Map, one Next Action, Do Not Do, and the final Human Confirmation Gate.

### Modes

`prepare` is the outgoing checkpoint. It captures objective, current state, verification gates, evidence, decisions, open questions, one safe next action, and a `Do Not Do` list. When the project is writable and persistence is permitted, it writes the same body to both `LATEST.md` and a timestamped history file.

`recover` is the incoming verification pass. It reads the handoff and current project assets, compares decision-critical claims, emits a structured `Recovery Verification Report`, classifies the state, and stops for explicit human confirmation. It does not execute the proposed action, repair conflicts, or manufacture missing evidence before confirmation.

`CONFLICT` wins over `INSUFFICIENT_EVIDENCE`: one reliable contradiction in a decision-critical claim makes the overall verdict `CONFLICT`. Missing or stale evidence without a direct contradiction is `INSUFFICIENT_EVIDENCE`.

### Usage

In ChatGPT Chat/Work after installing the plugin:

```text
@session-handoff prepare
@session-handoff recover
```

In Codex:

```text
$session-handoff prepare
$session-handoff recover
```

Natural-language requests work as well, for example: “Prepare a handoff before I switch conversations” or “Recover from `.handoff/LATEST.md`, verify the live state, and do not continue yet.”

### How it differs from similar skills

Public handoff skills commonly optimize for one of four goals: automatic compaction hooks, durable snapshots, one-shot files consumed by a startup hook, or provider-neutral transcript continuity. `session-handoff` chooses a narrower safety contract: verify the live state before continuation and preserve a human decision point.

Compared with hook-driven tools such as [context-handoff](https://github.com/djhyes/context-handoff), it has no automatic lifecycle hooks and therefore no background behavior to trust. Compared with durable snapshot skills such as [codex-session-handoff](https://github.com/lifang336/codex-session-handoff) and [codex-session-handoff-skill](https://github.com/Liu-Bot24/codex-session-handoff-skill), it adds explicit state comparison, gate separation, and verdict semantics. Compared with one-shot `handoff.md` flows such as [codex-skills/handoff](https://github.com/lhh666-6/codex-skills/tree/main/skills/handoff), it preserves history and never treats file consumption as verification. Compared with provider-neutral adapters such as [continuity-handoff](https://github.com/ciumbar/continuity-handoff), it focuses on evidence quality and authorization boundaries rather than runtime adapters.

### Non-goals

- It does not replay a complete conversation transcript.
- It does not resolve conflicts on the user’s behalf.
- It does not bypass permissions, approvals, releases, or external-message authorization.
- It does not run a watcher, daemon, automatic monitor, or background continuation.
- It does not treat an old test result, old commit, or informal approval-like text as current truth.

### Development and validation

Validate the bundled skill and plugin manifest before distribution. Behavioral validation should check observable outcomes: `prepare` produces a complete evidence-backed handoff; `recover` inspects live assets, classifies the state correctly, and stops after the report.
