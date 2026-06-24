# Hermes-Memory-OS Landing Page

<div align="center">

**会治理的 AI 记忆系统**

[项目主页](https://btnalit.github.io/) ·
[源码仓库](https://github.com/btnalit/Hermes-Memory-OS) ·
[快速开始](https://github.com/btnalit/Hermes-Memory-OS#quick-start)

</div>

---

## 项目简介

Hermes-Memory-OS 是面向长期运行 Hermes Agent 的 file-first 记忆与治理运行时。

它为 Hermes profile 提供可审计的持久记忆、受边界约束的认知模块、owner review、右脑表达反馈和运维监控证据，同时不接管 Hermes 的对话、传输或调度职责。

核心边界很清楚：

- Hermes 仍然是 host agent，负责对话、LLM 交互、cron、投递、路由、澄清、重试和平台适配。
- Memory-OS 提供 profile-local canonical files、SQLite 索引、稳定 action token、helper scripts、audit 和 monitor evidence。
- Memory-OS 不是独立聊天服务器、SaaS 记忆服务或通用执行器。

## 核心能力

- **文件优先记忆基座**：以 profile-local canonical files 作为主权数据源，避免不可检查的黑盒存储。
- **可重建索引**：使用 SQLite/FTS5 支撑全文搜索、预取和证据查询。
- **混合检索**：支持 FTS5、可选本地 embedding 向量相似度、可选 graph-layer edge traversal，并可降级到纯 FTS5。
- **工作记忆与结晶记忆分离**：working memory 不会直接变成 owner-approved crystallized memory。
- **Owner 可见治理闭环**：审批、拒绝、反馈和 bounded apply 都通过稳定 action token 进入状态机。
- **右脑表达反馈**：保留非任务化表达通道，并用反馈 ledger 追踪表达质量。
- **监控证据**：通过 runtime、cron、monitor 和 probe 输出可验证的 PASS/WARN/FAIL 信号。
- **安全 apply 路径**：只有具备 bounded runtime target、rollback、monitor fields 和显式 apply token 的 proposal kind 才能执行。

## 快速开始

安装到已有 Hermes profile：

```bash
git clone https://github.com/btnalit/Hermes-Memory-OS.git
cd Hermes-Memory-OS
HERMES_HOME=/root/.hermes bash scripts/install_memory_os.sh --yes --operational \
  --hindsight auto
```

`--operational` 是常规开源安装路径，会安装并启用：

- `memory_os` Hermes memory provider；
- `memory-os-agent-os` Hermes shell plugin；
- portable Memory-OS runtime modules；
- heartbeat runtime；
- cognitive-loop integration harness；
- active-closure Hermes cron onboarding。

保守安装，不启用 operational cron set：

```bash
HERMES_HOME=/root/.hermes bash scripts/install_memory_os.sh --yes --production-safe \
  --hindsight off
```

安装 helper 但不启用 recurring cron jobs：

```bash
HERMES_HOME=/root/.hermes bash scripts/install_memory_os.sh --yes --operational \
  --no-enable-owner-cron-onboarding
```

安装器不会重启 `hermes-gateway.service`。

## 调度模型

默认 operational preset 使用 `active-closure` Hermes cron profile，只创建或确认当前治理闭环需要的 jobs：

| Job | Deliver | Agent | Purpose |
| --- | --- | --- | --- |
| `memory-os-owner-review-digest` | owner channel | yes | 向 owner 发送 approval items 和真实 alerts |
| `memory-os-proposal-followups-opsgate` | `local` | no | 将 approved proposals 路由到 OpsGate/report-only follow-up |
| `memory-os-index-sync` | `local` | no | 保持 SQLite search index 与 canonical files 同步 |

Hermes 拥有 scheduler、transport、retry、origin/local routing 和 agent wording；Memory-OS 拥有 helper outputs、tokens、state transitions、audit 和 monitor fields。

## 监控面板

项目包含只读静态 monitor dashboard，默认开放在 `0.0.0.0:3693`。它只读取 bounded snapshot evidence，没有 approve、apply 或 send 控件。

生成并服务 live snapshot：

```bash
python scripts/memory_os_monitor_dashboard_snapshot.py \
  --hermes-home /root/.hermes \
  --profile main \
  --output monitor_dashboard/snapshot.generated.js

python scripts/serve_memory_os_monitor_dashboard.py \
  --host 0.0.0.0 \
  --port 3693 \
  --snapshot-hermes-home /root/.hermes \
  --snapshot-profile main \
  --snapshot-interval-seconds 60
```

## 验证

安装后可使用 Hermes CLI 验证：

```bash
HERMES_HOME=/root/.hermes hermes memory
HERMES_HOME=/root/.hermes hermes plugins list
HERMES_HOME=/root/.hermes hermes memory-os-agent-os status
HERMES_HOME=/root/.hermes hermes memory-os-agent-os doctor
HERMES_HOME=/root/.hermes hermes memory-os-agent-os modules status
```

期望关系：

```text
memory.provider = memory_os
plugins.enabled includes memory-os-agent-os
plugins.enabled does not include memory_os
```

`memory_os` 通过 `memory.provider` 选择，不应作为通用 Hermes plugin 启用。

## Owner 交互

Owner review 和 feedback 发生在正常 Hermes 对话频道中。owner 不应需要 root shell 才能审批 Memory-OS items。

示例 owner 指令：

```text
memory approve oa_<token>
memory reject oa_<token>
memory apply oa_<token>
memory feedback oa_<token> too_mechanistic
memory feedback oa_<token> like_expression
```

Hermes 负责理解 owner utterance、在歧义时追问，并调用结构化 Memory-OS review tool。Memory-OS 只通过稳定 token 和 owner action state machine 处理状态变化。

## 安全模型

Memory-OS 默认保持以下边界：

- 不从 Memory-OS 直接向外部平台发送消息；
- 不执行外部任务；
- 没有 owner-approved path 时不写身份信息；
- 不写入未批准的 crystallized memory；
- 不做未治理的 Hindsight export 或 raw-turn retain；
- cleanup 或 shadow-journal apply 需要单独 gate。

Canonical data 位于 Hermes profile 下。SQLite indexes 可重建。review、feedback 和 apply 操作都会审计。

## 架构边界

```text
Hermes agent
  owns conversation, LLM interaction, cron, delivery, origin/local routing,
  clarification, transport, retry, and channel adapters

Memory-OS provider
  owns profile-local canonical memory, indexes, prefetch, sync_turn,
  working memory, crystallized candidates, and audit

Memory-OS retrieval
  FTS5 full-text search
  -> optional vector similarity
  -> optional graph-layer edge traversal
  -> Reciprocal Rank Fusion union over active lanes

Memory-OS Agent OS shell
  exposes operator status, doctor, module, feedback, and review tools

OwnerActionProcessor
  is the state-changing entry point for approve, reject, feedback, allow,
  and bounded apply actions
```

## 仓库结构

```text
memory_os_agent/               Minimal Hermes compatibility surface
plugins/memory/memory_os/      Memory-OS provider and core services
plugins/memory-os-agent-os/    Hermes shell plugin and review tools
plugins/system/                Module contracts and coordination primitives
plugins/modules/               Portable cognition/governance/expression modules
scripts/                       Installer, monitor, cron helpers, validation
tests/                         Provider, module, installer, and monitor tests
docs/                          Public operator docs
```

普通用户应从 upstream 仓库的 `docs/quickstart.md` 和 `docs/configuration.md` 开始。

## 开发与检查

```bash
python -m pip install -e ".[dev]"
python -m pytest -q
git diff --check
```

修改 scheduler、owner review、feedback、installer、module closure 或 live monitor 行为前，需要先确认 owning interface，并运行相关检查。

## 许可证

MIT License. See [btnalit/Hermes-Memory-OS](https://github.com/btnalit/Hermes-Memory-OS).
