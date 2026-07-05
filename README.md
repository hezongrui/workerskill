# 中文软件工程 Skills

这是一组面向日常软件开发的 Claude Code skills，重点覆盖需求澄清、架构设计、开发规划和代码审查。
每个 skill 只对应一个具体场景，按需使用，不需要完整走完全流程。

## 推荐工作流

### 1. 标准开发主流程

适合：新需求、较完整功能、需要文档留痕的日常开发

1. `plan-prd`：把需求整理成可执行的需求文档。
2. `design-architect-system`：根据业务场景和代码现状做架构设计。
3. `plan-dev`：确定模块、文件边界和开发顺序。
4. `code-review`：审查 diff / PR / 本地改动，优先找真实风险。

### 2. 轻量开发流程

适合：小需求、小改动、局部重构、已有需求和边界比较清楚

1. `plan-dev`
2. `code-review`

### 3. bug 修复流程

适合：线上 bug、已知缺陷、回归问题

1. `bug-plan`：先把现象、复现、范围、期望结果和修复边界整理清楚。
2. `bug-investigation`：配合用户逐步定位根因，每轮记录验证动作和结果。
3. `code-review`

### 4. 新人上手流程

适合：快速理解现有项目，不直接改代码

1. `onboard-architecture`
2. `onboard-modules`
3. `onboard-focus`

## Skill 说明

| Skill | 使用场景 | 默认输出 |
| --- | --- | --- |
| `plan-discover` | 需求还不清楚，需要先聊清目标、边界和成功标准 | `docs/specs/` |
| `plan-prd` | 需求大致明确，需要整理成 PRD | `docs/prd/` |
| `design-architect-system` | 新项目架构设计、老项目增量设计、模块拆分、技术方案 | `docs/architect/` |
| `bug-plan` | 修 bug 前先整理问题现象、复现步骤、影响范围和修复边界 | `docs/bugs/` |
| `bug-investigation` | 配合用户逐步定位 bug 根因，每轮验证都记录操作和结果 | `docs/bugs/` |
| `plan-dev` | 根据需求或架构文档确定开发顺序和文件边界 | `docs/plans/` |
| `code-review` | 审查 diff、PR 或本地改动 | `docs/reviews/` |
| `onboard-architecture` | 新人快速了解项目整体架构 | `docs/onboarding/architecture.md` |
| `onboard-modules` | 分析各模块职责和学习优先级 | `docs/onboarding/` |
| `onboard-focus` | 深入分析重点模块 | `docs/onboarding/modules/` |

## 常见用法

1. 新项目：
   `plan-prd` -> `design-architect-system` -> `plan-dev` -> `code-review`

2. 老项目加功能：
   `plan-prd` -> `design-architect-system` -> `plan-dev` -> `code-review`

3. 修 bug：
   `bug-plan` -> `bug-investigation` -> `code-review`

4. 新人上手项目：
   `onboard-architecture` -> `onboard-modules` -> `onboard-focus`

5. 需求还不清楚：
   `plan-discover` -> `plan-prd` -> 后续按主流程继续

## 使用原则

1. 优先使用项目已有规范，不另起一套流程。
2. 小需求走轻流程，大需求再走 PRD 和架构文档。
3. 架构设计以业务场景和约束为依据，避免过度设计。
4. 代码审查优先发现真实风险：bug、回归、缺测试、安全、并发、资源和性能问题。
5. 文档只写能帮助开发和审查的信息。
