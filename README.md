# 中文软件工程 Skills

这是一组面向日常软件开发的 Claude Code skills，重点覆盖需求澄清、架构设计、开发规划、测试驱动、测试计划、代码审查、交付收口、失败分诊和错误沉淀。
每个 skill 只对应一个具体场景，按需使用，不需要完整走完全流程。

## 推荐工作流

### 1. 标准开发主流程

适合：新需求、较完整功能、需要文档留痕的日常开发

1. `plan-prd`：把需求整理成可执行的需求文档。
2. `design-architect-system`：根据业务场景和代码现状做架构设计。
3. `plan-dev`：确定模块、文件边界和开发顺序。
4. `test-driven-development`：按优先级做测试先行，实现功能、修 bug 或重构。
5. `test-plan-design`：补齐自动测试和手动测试用例矩阵。
6. `code-review`：审查 diff / PR / 本地改动，优先找真实风险。
7. `pr-delivery`：整理测试证据、剩余风险、commit message 和 PR 描述。

### 2. 轻量开发流程

适合：小需求、小改动、局部重构、已有需求和边界比较清楚

1. `plan-dev`
2. `test-driven-development`
3. `code-review`
4. `pr-delivery`

### 3. bug 修复流程

适合：线上 bug、已知缺陷、回归问题

1. `test-driven-development`：先写失败测试复现问题，再修复。
2. `code-review`
3. `pr-delivery`
4. `error-retrospective`

### 4. CI / 构建失败流程

适合：编译挂、链接挂、单测挂、环境挂、flake

1. `ci-failure-triage`
2. `code-review`
3. `pr-delivery`
4. `error-retrospective`

### 5. 新人上手流程

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
| `design-ui-general` | 需要设计界面布局、页面结构、Qt/Web 交互 | `docs/design/` |
| `plan-dev` | 根据需求或架构文档确定开发顺序和文件边界 | `docs/plans/` |
| `test-driven-development` | 写新功能、修 bug、重构、补测试 | 代码和测试 |
| `test-plan-design` | 为模块或功能设计测试用例矩阵 | `docs/test-plans/` |
| `code-review` | 审查 diff、PR 或本地改动 | `docs/reviews/` |
| `pr-delivery` | 开发完成后整理提交材料、测试证据和风险 | `docs/delivery/` |
| `ci-failure-triage` | 构建、测试、流水线失败时快速归因和分诊 | `docs/triage/` |
| `error-retrospective` | 沉淀 bug、事故、CI 失败和 review 问题的复盘 | `docs/retrospectives/` |
| `onboard-architecture` | 新人快速了解项目整体架构 | `docs/onboarding/architecture.md` |
| `onboard-modules` | 分析各模块职责和学习优先级 | `docs/onboarding/` |
| `onboard-focus` | 深入分析重点模块 | `docs/onboarding/modules/` |
| `guide-tech` | 讲解技术概念、架构原理或代码逻辑 | `docs/guide-tech/` |

## 常见用法

1. 新项目：
   `plan-prd` -> `design-architect-system` -> `plan-dev` -> `test-driven-development` -> `test-plan-design` -> `code-review` -> `pr-delivery`

2. 老项目加功能：
   `plan-prd` -> `design-architect-system` -> `plan-dev` -> `test-driven-development` -> `test-plan-design` -> `code-review` -> `pr-delivery`

3. 修 bug：
   `test-driven-development` -> `code-review` -> `pr-delivery` -> `error-retrospective`

4. CI / 流水线失败：
   `ci-failure-triage` -> `code-review` -> `pr-delivery` -> `error-retrospective`

5. 新人上手项目：
   `onboard-architecture` -> `onboard-modules` -> `onboard-focus`

6. 需求还不清楚：
   `plan-discover` -> `plan-prd` -> 后续按主流程继续

7. 没有界面需求：
   跳过 `design-ui-general`

## 使用原则

1. 优先使用项目已有规范，不另起一套流程。
2. 小需求走轻流程，大需求再走 PRD 和架构文档。
3. 架构设计以业务场景和约束为依据，避免过度设计。
4. TDD 只用于有明确可验证行为的开发任务，不为测试而测试。
5. 代码审查优先发现真实风险：bug、回归、缺测试、安全、并发、资源和性能问题。
6. 交付收口要写真实测试证据、剩余风险和未覆盖点。
7. CI 失败先看第一有效失败点，不被连锁报错带偏。
8. 错误复盘要反哺测试、审查、交付或 CI，不写空泛总结。
9. 文档只写能帮助开发、测试、审查和交付的信息。
