---
name: plan-execution
description: |
  触发关键词：拆解计划
  触发条件：已有规划或设计，要拆成每步2-5分钟的可执行操作时。
---

# 写执行计划

## 一句话原则
**把实现拆成 2-5 分钟一个的小任务，每个任务都写清：改哪个文件、怎么测、预期结果。**

## 前置条件
开始拆任务前，先检查并阅读以下已有文档（如果有的话）：
- `docs/plans/dev-[project-name].md`（`plan-dev` skill 的输出）
- `docs/architect/` 下的架构设计文档
- 相关的 PRD 或需求文档

**同时必须检查：**
- 项目是否已有统一的日志模块。如果没有，**第一个任务必须是搭建日志模块**，确保后续每个模块的测试和运行都有完整日志输出，便于定位问题。

不要凭空拆任务，要基于已有设计拆分。

## 保存路径
`docs/plans/YYYY-MM-DD-<功能名>.md`
（用户指定了别的路径就按用户的来）

## 计划头部格式
每个计划文档必须以这段开头：

```markdown
# [功能名] 实现计划

> **对于自动执行：** 推荐用 `dev-subagent-driven` skill 一步一步执行，也可以在当前会话里按批次执行。下面每一步都用 `- [ ]` checkbox 语法，方便跟踪进度。

**目标：** 一句话说明这个计划要做出什么

**整体思路：** 2-3 句话讲讲怎么做

**技术栈：** 用到的主要技术/库

---
```

## 任务拆分粒度

**每个步骤只做一件事，耗时 2-5 分钟。**

好例子：
- 写失败的测试
- 跑测试确认它失败
- 写最小实现让测试通过
- 跑测试确认通过
- 提交 commit

## 任务格式模板

```markdown
### 任务 N：[组件名]

**涉及文件：**
- 新建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 测试：`tests/exact/path/to/test.py`

**执行方式**：`test-driven-development`（红绿循环）/ 直接实现 / 其他 skill

- [ ] **步骤 1：写失败的测试**

```python
def test_具体行为():
    result = function(input)
    assert result == expected
```

- [ ] **步骤 2：跑测试，确认失败**

运行：`pytest tests/path/test.py::test_name -v`
预期：FAIL，提示 "function not defined"

- [ ] **步骤 3：写最小实现**

```python
def function(input):
    return expected
```

- [ ] **步骤 4：跑测试，确认通过**

运行：`pytest tests/path/test.py::test_name -v`
预期：PASS



## 注意事项

- **用精确文件路径**：不要写“改那个文件”，要写完整路径
- **尽量给完整代码**：不要只写“加验证”
- **给出精确命令和预期输出**
- **标注执行方式**：每个任务开头明确标注采用的开发方式或 skill（如 `test-driven-development`、`dev-subagent-driven`、直接实现等）
- **引用相关 skill**：比如执行此步骤时请遵循 `test-driven-development` skill 的红绿循环
- **日志先行**：每个模块拆解时，若项目尚无日志模块，必须将「搭建日志模块」列为该模块的第一个任务；日志输出需覆盖 DEBUG/INFO/WARN/ERROR，包含时间戳与模块名
- **DRY**：不要重复
- **YAGNI**：不要超前设计
- **频繁提交**：每完成一个小任务就 commit
- **进度跟踪**：在当前会话执行时，使用当前环境可用的任务/进度跟踪工具（如 `TaskCreate`、`TaskUpdate` 等）跟踪每个小任务的完成状态

## 计划评审

写完完整计划后，必须：

1. 用 `Agent` 工具派遣一个独立的 reviewer subagent 审计划，检查：任务是否过粗/过细、依赖关系是否正确、是否有遗漏的测试步骤
2. 有问题 → 改 → 再审，最多 3 轮
3. 通过后，交给用户执行


