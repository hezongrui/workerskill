# Claude Code Skills - 中文软件工程工作流

一套覆盖需求、设计、规划、开发、项目上手等全流程的 Claude Code 自定义技能。

每个技能对应一个具体的工程场景，按需调用，不必走完完整流程。

---

## 场景速查：什么时候调用哪个技能

### 需求阶段

| 场景 | 调用技能 | 输出 |
|------|---------|------|
| 只有一个模糊想法，不确定要做什么 | `plan-discover` | 设计文档 `docs/specs/YYYY-MM-DD-<主题>-design.md` |
| 需求大致清楚，要整理成结构化文档 | `plan-prd` | 需求文档 `docs/prd/<需求名>.md` |

### 设计阶段

| 场景 | 调用技能 | 输出 |
|------|---------|------|
| 需要技术选型、系统架构、模块拆分 | `design-architect-system` | 架构文档 `docs/architect/` |
| 有界面，需要设计布局和交互 | `design-ui-general` | UI 设计文档 `docs/design/<模块>-ui-design.md` |

### 规划阶段

| 场景 | 调用技能 | 输出 |
|------|---------|------|
| 确定文件/模块的开发顺序和依赖关系 | `plan-dev` | 开发规划 `docs/plans/dev-<项目名>.md` |
| 已有设计，要拆成具体可执行的步骤 | `plan-execution` | 执行计划 `docs/plans/YYYY-MM-DD-<功能名>.md` |

### 开发阶段

| 场景 | 调用技能 | 说明 |
|------|---------|------|
| 有书面计划，想让 Agent 自动逐个任务执行 | `dev-subagent-driven` | 多 Agent 并行/串行执行，自动审查 |
| 写新功能、修 bug、重构、改行为 | `test-driven-development` | 先写测试再写实现，红绿循环 |

### 项目上手阶段（新人进项目）

| 场景 | 调用技能 | 输出 |
|------|---------|------|
| 刚进项目，快速了解整体架构 | `onboard-architecture` | 架构全景 `docs/onboarding/architecture.md` |
| 有了整体架构，细化各模块优先级 | `onboard-modules` | 模块分析 + 优先级总览 `docs/onboarding/` |
| 确定了重点模块，深挖核心逻辑 | `onboard-focus` | 重点清单 `docs/onboarding/modules/<模块名>-key-points.md` |

### 讲解/知识沉淀

| 场景 | 调用技能 | 输出 |
|------|---------|------|
| 需要向团队/新人讲解某个技术概念 | `guide-tech` | 讲解文档 `docs/guide-tech/<topic-name>.md` |

---

## 完整工作流（按需裁剪）

以下是一条完整的软件工程流程，实际使用时根据项目规模和阶段选择性调用：

```
[需求]
  |
  ├─ 想法模糊 ────────────────> plan-discover ───────┐
  |                                                   |
  └─ 需求大致清楚 ────────────> plan-prd ─────────────┤
                                                      v
[设计]                                               docs/prd/
  |
  ├─ 需要技术选型/架构设计 ───> design-architect-system
  |                              |
  |                              v
  |                           docs/architect/
  |
  └─ 有界面要设计 ────────────> design-ui-general
                                 |
                                 v
                              docs/design/
[规划]
  |
  ├─ 确定开发顺序 ────────────> plan-dev ─────────────┐
  |                                                   |
  └─ 拆成可执行步骤 ──────────> plan-execution ───────┤
                                                      v
[开发]                                               docs/plans/
  |
  ├─ 自动执行计划 ────────────> dev-subagent-driven
  |                              |
  |                              └─ 内部调用 test-driven-development
  |
  └─ 单独写功能/修 bug ───────> test-driven-development

[项目上手]（与开发阶段独立，用于新人熟悉代码库）
  |
  ├─ 先看全局 ────────────────> onboard-architecture
  |                              |
  |                              v
  |                           docs/onboarding/architecture.md
  |
  ├─ 再看模块 ────────────────> onboard-modules
  |                              |
  |                              v
  |                           docs/onboarding/module-priority-overview.md
  |                           docs/onboarding/modules/<模块名>-module-analysis.md
  |
  └─ 最后深挖 ────────────────> onboard-focus
                                 |
                                 v
                              docs/onboarding/modules/<模块名>-key-points.md
```

### 常见裁剪方式

- **小型脚本/工具**：`plan-prd` -> `plan-execution` -> `test-driven-development`
- **纯后端 API 项目**：跳过 `design-ui-general`
- **已有明确需求的项目**：跳过 `plan-discover`
- **老项目加功能**：`design-architect-system`（增量模式）-> `plan-dev` -> `dev-subagent-driven`
- **团队新人上手**：`onboard-architecture` -> `onboard-modules` -> `onboard-focus`

---

## 目录结构

```
.
├── guide-tech/SKILL.md                  # 技术讲解
├── test-driven-development/
│   ├── SKILL.md                          # TDD 主技能
│   └── references/                       # 各语言参考
│       ├── cpp.md
│       ├── go.md
│       ├── python.md
│       ├── python-backend-patterns.md
│       ├── typescript-frontend.md
│       ├── qml.md
│       ├── qwidget.md
│       └── testing-anti-patterns.md
├── plan-prd/SKILL.md                     # 需求分析 / PRD
├── plan-discover/SKILL.md                # 需求梳理 / 头脑风暴
├── dev-subagent-driven/SKILL.md          # 多 Agent 驱动开发
├── design-architect-system/
│   ├── SKILL.md                          # 架构设计主技能
│   └── references/                       # 架构模式模板
│       ├── quick.md
│       ├── standard.md
│       ├── enterprise.md
│       └── increment.md
├── plan-execution/SKILL.md               # 执行计划拆解
├── onboard-focus/SKILL.md                # 单模块深挖
├── onboard-architecture/SKILL.md         # 项目架构扫描
├── onboard-modules/SKILL.md              # 模块分析
├── design-ui-general/SKILL.md            # UI 设计
├── plan-dev/SKILL.md                     # 开发规划
└── README.md                             # 本文档
```

---

## 使用方式

1. 将本仓库内容复制到 Claude Code 技能目录：
   - Windows: `%USERPROFILE%\.claude\skills`
   - macOS/Linux: `~/.claude/skills`

2. 重启 Claude Code，或在对话中使用 `/技能名` 触发对应技能。

3. 每个技能文件头部包含触发关键词和触发条件，Claude Code 会根据对话内容自动匹配。

---

## 注意事项

- `design-architect-system` 中引用的 Tavily 搜索工具名与 Claude Code 实际 MCP 工具名存在差异，技能内已设置回退机制（回退到内置 `WebSearch`/`WebFetch`）。
- `dev-subagent-driven` 期望 Agent 内部嵌套调用 `test-driven-development` skill，但 Agent 内部无法直接调用 `Skill` 工具。实际使用时建议将 TDD 规范内联到实现者 Agent 的 prompt 中。
- `references/` 子目录下的文件在 skill 触发时不会自动加载到上下文中，如需引用请手动提供。

---

