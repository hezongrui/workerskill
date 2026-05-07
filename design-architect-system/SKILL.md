---
name: design-architect-system
description: |
  触发关键词：系统架构、模块设计、架构设计、架构文档、用什么技术做、XX系统怎么设计
  触发条件：新项目架构设计、老项目加功能/模块扩展、需要技术选型、多模块系统需要模块拆分。
  输出路径：docs/architect/
---

# 架构设计协议

## 核心限制

1. **只写架构，不写实现代码**（无函数体、SQL、可运行代码）
2. **每个模块一句话描述职责**
3. **必须给出文件夹层级结构，并标注每个层级的功能**（让开发者一眼看到架构对应的目录映射）
4. **依赖方向单向**，禁止循环依赖
5. **技术选型必须调研验证**，给出"为什么不用其他方案"
     - **优先使用 Tavily MCP**：简单查证用 `tavily_search`，深度调研用 `tavily_research`，获取网页详情用 `tavily_extract`
     - **若 Tavily 调用失败或不可用**，回退到内置网络搜索工具；**若内置搜索亦不可用（如网络受限），停止搜索，改用训练知识库完成决策**
6. **风险项格式**：触发条件 + 影响 + 应对 + 回退
7. **信息不足时显式列出假设前提**
8. **不要默认 Quick 模式**：需要长期维护的产品即使阶段小也应选 Standard，只有一次性脚本/验证原型才选 Quick
9. **禁止跨阶段合并**：若 PRD 已明确分阶段（阶段 1/2/3...），**必须先生成该阶段的独立架构文档**。一份架构文档只描述一个阶段。
10. **禁止跨层合并**：若一个阶段同时包含多个逻辑层（如服务端/客户端、后端/前端、引擎/UI、算法层/应用层），**每层单独输出一份架构文档**。禁止把不同层的模块混在同一份文档里。

---

## 阶段与层级确认规则（强制执行）

开始写架构前，**必须先读取对应的 PRD**。

### 情况 A：PRD 已分阶段

若 PRD 中出现了"阶段 1 / 阶段 2 / 阶段 3"或类似分期描述：

1. **停止**，不要直接写大而全的文档。
2. **使用 `AskUserQuestion` 询问用户**：
   > "PRD 已将需求分为阶段 1/2/3，本次要生成哪个阶段的架构文档？"
3. **用户确认阶段 N 后**，进一步判断该阶段是否涉及多个逻辑层（如后端 + 前端、C++ 引擎 + QML UI）。
4. **若涉及多层**：使用 `AskUserQuestion` 再次确认：
   > "阶段 N 同时包含 [后端/前端/...] 多层架构，本次要生成哪一层的架构文档？"
5. **用户确认具体层后**，只输出该层对应的架构文档。

### 情况 B：PRD 未分阶段

直接按单份完整架构文档输出。但若 PRD 仍明显包含多个逻辑层，建议主动询问用户是否按层拆分。

### 绝对禁止
- 把阶段 1 和阶段 2 的模块合并到同一个文档里。
- 把后端模块和前端模块合并到同一个文档里。
- 在一份文档中写"后端用 FastAPI，前端用 React，UI 用 Qt QML"这类混层内容。

## 模式选择

**用户已明确** → 按用户要求  
**用户未明确** → AI 根据**项目整体特征**自主决策，**不要只看当前阶段的功能数量**：

| 模式 | 决策条件 |
|------|----------|
| **Quick** | 一次性脚本/内部小工具/验证原型；功能极简；无长期维护需求；单文件或单模块即可解决 |
| **Enterprise** | 平台级/多租户、需高可用(99.99%+)、大数据量、多系统集成、明显暗示多团队协作 |
| **Standard** | 需要持续迭代维护的产品；有前后端或多模块；即使当前阶段功能少，但项目整体会演进扩展；绝大多数项目默认选此 |

**关键提醒**：PRD 每个阶段可能只有 2-3 个功能，但模式选择要看**项目整体复杂度和演进空间**。一个会被长期维护、会加前端/后端/新模块的产品，即使阶段 1 很小，也应选 **Standard**，不要降级为 Quick。

---

## 技术选型决策

**用户已指定技术栈** → 直接使用，优先用 Tavily MCP 验证是否合适；Tavily 不可用时回退到内置网络搜索工具；若两者均不可用（如网络受限），停止搜索，改用训练知识库完成验证  
**用户未指定** → AI根据需求特征自主决策：

```
需求特征分析 → 从训练知识提取 2-4 个候选技术方向
    ↓
优先用 Tavily MCP 验证 → "[技术] [场景] best practice" 查现状
    ↓ Tavily 不可用则回退到内置网络搜索工具；两者均不可用时停止搜索
决策输出 → 推荐方案 + 1-2备选 + 引用来源（若搜索均未完成则注明基于训练知识） + "为什么不用其他方案"
```

---

## 项目类型判断（强制执行）

开始架构设计前，**必须先判断用户是在已有项目上增量，还是从零搭建新项目**：

### 情况 A：老项目加功能（增量架构）

若用户提到"在现有项目上增加"、"基于已有代码扩展"、"给 XX 系统加模块"，或工作区中已存在大量代码文件：

1. **先扫描现有项目**：使用 `list_dir`、`search_codebase`、`read_file` 等工具读取现有目录结构、技术栈（`package.json`、`pyproject.toml`、`requirements.txt`、`Cargo.toml` 等）、核心模块
2. **识别现有架构模式**：判断是 Quick/Standard/Enterprise 中的哪种风格
3. **基于现有架构设计增量方案**：新增模块必须融入现有目录结构和依赖规则，不能另起炉灶
4. **明确标注**：
   - `【新增】`：新建的文件/目录
   - `【修改】`：需要改动的现有文件
   - `【复用】`：直接复用的现有模块

### 情况 B：新项目从零搭建

若工作区为空或用户明确表示"从零开始"、"新建项目"：

直接按本 skill 的标准流程输出完整架构文档。

---

## 执行流程

> **参考文件清单**（按项目类型选择对应文件读取）：
> - 新项目 Quick：`references/quick.md`
> - 新项目 Standard：`references/standard.md`
> - 新项目 Enterprise：`references/enterprise.md`
> - 老项目增量（Increment）：`references/increment.md`

### 新项目流程

1. **收集信息**：系统目标、核心场景（2-5个）、技术约束
2. **技术调研**：优先使用 Tavily MCP 验证 2-4 个候选方案；Tavily 不可用时回退到内置网络搜索工具；若两者均不可用（如网络受限），停止搜索，改用训练知识库进行对比决策
3. **选择模式**：按上表信号确定 Quick/Standard/Enterprise，**基于项目整体需求而非当前阶段功能数量**
4. **读取模式文件**：执行系统命令读取（Windows: `type references\[mode].md`，Unix: `cat references/[mode].md`）
5. **按模式文件执行**：使用其中的模板输出架构文档到 `docs/architect/`

### 老项目增量流程

1. **扫描现有代码库**：读取目录结构、核心配置文件、现有模块划分
2. **识别现有架构**：判断当前项目属于 Quick/Standard/Enterprise 哪种风格，用了什么技术栈
3. **分析增量需求**：明确新增功能需要哪些模块、接口、数据流
4. **读取增量模板**：执行系统命令读取 `references/increment.md`
5. **设计增量方案**：
   - 新增模块的职责和目录位置
   - 与现有模块的依赖关系（禁止循环依赖）
   - 需要修改的现有文件和接口
6. **输出增量架构文档**：使用 `docs/architect/architect-[mode]-[project]-increment-[feature].md` 命名

---

## 输出文件命名

### 单阶段、单层项目（未分阶段且只有一层）
- Quick: `docs/architect/architect-quick-[project].md`
- Standard: `docs/architect/architect-standard-[project].md`
- Enterprise: `docs/architect/architect-enterprise-[project].md`

### 分阶段项目（PRD 已分期）
- 阶段 N: `docs/architect/architect-[mode]-[project]-phase[N]-[layer].md`
- 示例：
  - 后端：`docs/architect/architect-standard-industry-report-generator-phase1-backend.md`
  - 前端工程：`docs/architect/architect-standard-industry-report-generator-phase2-frontend.md`
  - C++ 引擎：`docs/architect/architect-standard-myapp-phase1-engine.md`
  - Qt QML UI：`docs/architect/architect-standard-myapp-phase1-ui.md`

### 老项目增量架构
- `docs/architect/architect-[mode]-[project]-increment-[feature].md`
- 示例：
  - `docs/architect/architect-standard-ecommerce-increment-payment.md`
  - `docs/architect/architect-quick-crawler-increment-scheduler.md`

**阶段与层级聚焦原则**：
- 当前文档**仅包含**用户确认的【阶段 + 层】所需的模块、接口、数据流和风险项。
- 对该阶段的其他层、或后续阶段的功能，**一字不提**。

---

## 与其他 skill 的协作边界

| 范围 | 负责 Skill | 输出路径 | 核心内容 |
|------|-----------|----------|----------|
| **工程架构**（任何技术栈的模块、数据流、接口契约、目录结构） | `design-architect-system`（本 skill） | `docs/architect/` | 技术选型、模块划分、依赖关系、数据流、存储设计、API/RPC/信号契约 |
| **界面视觉与交互设计**（布局图、配色、Tokens、动效） | `design-ui-general` | `docs/design/` | 页面结构、视觉层级、组件尺寸、颜色/字体 Tokens、交互状态 |

**关键原则**：
- `design-architect-system` 是**工程架构**文档，描述"系统怎么搭、模块怎么拆、数据怎么流"。
- `design-ui-general` 是**视觉设计**文档，描述"界面长什么样、用户怎么交互"。
- 两者互补，不是替代关系。

**执行顺序建议**：
1. 用户确认阶段 N 和具体层后，由本 skill 输出该层的工程架构文档。
2. 若该层涉及界面（如 Web 前端、Qt QML、Android/iOS UI），建议先在 `design-ui-general` 中完成界面设计，再在本 skill 中把 UI 结构映射为工程模块（如组件目录、ViewModel、Store 设计）。
3. 若用户没有界面设计需求，直接由本 skill 输出即可。

