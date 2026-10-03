# ruan-requirement-discovery 2.0

## 1. 定位

`ruan-requirement-discovery` 是一个面向软件项目**下一版本需求发现与需求分析**的长期编排 Skill。

它负责把零散、重复、冲突、过期、不完整的用户反馈，与当前项目代码和真实业务背景结合，逐步整理为：

- 可追溯的反馈索引；
- 已核实的当前技术事实；
- 必要的业务与使用背景事实；
- 可落地的需求项；
- 用户确认的重要决策；
- 跨需求冲突与全局约束；
- 可直接交给后续迭代规划阶段消费的 `planning-ready` 需求。

整体链路：

> **原始反馈 → 需求发现与核实 → 需求细化与决策 → planning-ready → 迭代规划 → 开发实现**

本 Skill 只负责前半段。

它不是开发 Skill，不负责实现需求，也不是代码版本、一致性或任务调度系统。

# 2. 核心目标

建立一套：

> **基于真实反馈、真实代码和真实业务背景，可以在有限上下文下分轮推进、跨会话恢复，并稳定产出下一版本需求输入的轻量需求发现流程。**

主要能力：

1. 接收未经整理的反馈材料；
2. 渐进建立稳定反馈索引；
3. 读取当前代码核实现状；
4. 区分 Technical Fact、Context Fact 和 Decision；
5. 将反馈逐步细化为需求；
6. 只在必要时向用户追问；
7. 识别局部需求之间的共同问题和冲突；
8. 持久化重要事实、决策和恢复状态；
9. 判断需求是否达到 `planning-ready`；
10. 向后续迭代规划提供稳定输入。

# 3. 硬边界

本 Skill：

- 不实现需求；
- 不修改业务代码；
- 不进入正式开发；
- 不自动执行后续迭代计划；
- 不要求用户提前整理反馈；
- 不把可以通过代码核实的问题转嫁给用户；
- 不迷信 README、ADR、旧需求文档或反馈描述；
- 不把 AI 猜测出的用户需求当成正式需求；
- 不建立数据库、复杂状态机、锁、任务调度器或一致性系统；
- 不负责 Git 分支、commit、发布版本或并发修改管理。

允许搜索、读取和分析代码，但目的仅限：

> **核实现状并辅助需求判断。**

# 4. Stable Input Contract

本 Skill 基于**稳定输入**工作。

用户交给 Skill 的：

- 项目代码；
- 用户反馈；
- 需求资料；

默认就是：

> **本次分析应依据的当前最新、有效输入，或用户明确选择的固定快照。**

Skill 不负责判断：

- Git 分支；
- commit；
- 快照生成方式；
- 发布版本；
- 输入是否正在被其他进程修改；
- 新旧代码之间如何同步。

使用约定：

> 在一次 Requirement Discovery 工作持续期间，不应同时修改 Skill 当前分析所依赖的项目代码和原始反馈资料。

如果仍需继续修改原项目，用户可以自行提供项目副本或固定快照。

如果输入发生实质变化：

> 用户明确告知 Skill“输入已更新”，Skill 重新核实受影响范围。

Skill 不主动监控输入变化。

## 4.1 输入更新后的处理

收到“输入已更新”后：

1. 先询问或使用用户已经提供的变更范围，例如模块、路径、功能或反馈来源；
2. 通过 REQ 中已有的代码路径、符号名、`touches`、`source_fb` 等定位受影响需求；
3. 重新核实这些需求。

如果用户无法说明变化范围：

> 优先重新核实当前 `planning-ready` 的需求，因为这些需求会被下游规划消费。

不引入 Git diff 或其他版本检测机制。

# 5. 一个 Skill，薄主流程

本能力保持为一个：

> ```
> ruan-requirement-discovery
> ```

不拆成多个独立 Skill。

反馈、需求、用户决策、跨需求冲突和恢复状态共同构成同一个长期任务。

内部采用：

> **薄** `**SKILL.md**` **+ 按需 References**

而不是把所有细节都放进主 Skill。

# 6. 主 Skill 与 References 的职责边界

采用唯一原则：

> **SKILL.md 负责“是什么、什么时候做、做到什么程度算完成”；References 负责“具体怎么做、字段和格式是什么”。**

同一规则不得同时在两处完整维护。

## SKILL.md 保留

- Skill 定位与硬边界；
- Stable Input Contract；
- 三类信息及其权威规则；
- 固定恢复入口；
- 七步主流程；
- 每一步 Completion Criteria；
- Decision Frontier 的原则；
- 全局复盘触发条件；
- planning-ready Contract；
- planning-ready 失效原则；
- 文档写入顺序；
- 收束规则；
- Step → Reference 加载映射。

## References 保留

- 反馈具体整理方式；
- 代码核实方法；
- REQ 字段和格式；
- `touches` 使用规范；
- Decision / Context Fact 记录格式；
- CON 记录格式；
- 各类文档最小模板和示例。

# 7. 三类信息

## 7.1 Technical Fact

能够通过当前项目证据核实的信息。

主要来源：

- 当前代码；
- 配置；
- 测试；
- 数据结构；
- API 契约；
- 用户提供的运行证据。

原则：

> **Technical facts are discovered.**

Technical Fact 描述的是：

> **用户本次提供给 Skill 的代码快照。**

## 7.2 Context Fact

代码本身无法可靠确定，但用户或可靠材料能够提供的真实业务、使用或运行背景。

例如：

- 现场实际有哪些角色；
- 某个流程真实如何使用；
- 某数据来自哪个外部系统；
- 线上环境当前仍采用什么行为；
- 真实业务范围是什么。

它回答：

> **现实环境是什么。**

如果只影响一个 REQ：

> 直接写入对应 REQ。

如果会影响多个 REQ：

> 写入 `CONTEXT.md`，生成稳定 `CTX-xxx` ID。

例如：

```
CTX-003
```

REQ 引用共享 Context Fact 时必须引用对应 `CTX ID`。

如果某个原本只存在于单个 REQ 的 Context Fact 后来发现会影响多个需求：

1. 将正文迁入 `CONTEXT.md`；
2. 分配 `CTX-xxx`；
3. 原 REQ 中只保留该 ID 的引用。

不维护两份正文。

## 7.3 Decision

存在多个都与当前证据相容的方向，而不同选择会改变下一版本需求时，需要用户决定。

例如：

- 是否保留旧行为；
- 本次版本纳入什么范围；
- 默认行为是什么；
- 两种合理业务规则选择哪一种。

原则：

> **Decisions are elicited.**

AI 可以提供默认推荐：

> **AI 推荐 ≠ 用户 Decision。**

# 8. 信息作用域

Technical Fact 和 Context Fact 可能同时成立。

例如：

- 当前代码快照中已经修复某问题；
- 用户说明生产环境仍运行旧行为。

这不是天然冲突。

Technical Fact 描述：

> 当前提供代码快照。

Context Fact 描述：

> 对应真实业务或运行环境。

只有两项事实在同一作用域内确实无法同时成立时，才进入进一步核实。

# 9. 默认输出目录

用户显式指定需求发现目录：

> 使用用户指定目录。

未指定时：

> 在反馈目录旁创建确定性的默认目录：

```
<反馈目录名>-requirement-discovery/
```

例如：

```
project/
├── user-feedback/
└── user-feedback-requirement-discovery/
```

后续给出相同反馈目录即可定位默认输出目录。

如果用户采用非默认目录：

> 后续新会话继续任务时由用户再次提供该目录。

不增加额外路径注册机制。

# 10. 固定运行文档

```
<requirement-discovery-dir>/
├── ENTRY.md
├── FEEDBACK_INDEX.md
├── CONTEXT.md
├── DECISIONS.md
├── CONFLICTS.md
├── CURRENT_QUESTIONS.md
└── requirements/
    ├── REQ-001.md
    ├── REQ-002.md
    └── ...
```

固定核心文件，避免不同会话重新发明结构。

# 11. ENTRY.md

`ENTRY.md` 是跨会话恢复的唯一入口。

应带稳定标识，例如：

```
skill: ruan-requirement-discovery
```

只维护：

- 任务目标；
- 反馈目录；
- 输出目录；
- 代码库根目录；
- 当前工作单元；
- 当前阻塞；
- 当前待回答问题；
- 当前需要关注的 CON；
- 下一步；
- 必要文档指针。

原则：

> **状态 + ID + Pointer**

ENTRY 不复制完整需求、代码事实、Context Fact 或 Decision。

不维护容易漂移的统计汇总。

# 12. 恢复与初始化

解析输出目录后：

### 如果该目录存在 `ENTRY.md`

> 进入恢复流程。

先读 ENTRY，再只读取当前任务点名的必要内容。

### 如果不存在 `ENTRY.md`

> 进入初始化流程。

初始化必须幂等：

- 已存在且非空的运行文件不得覆盖；
- 缺失的文件再创建；
- 已有内容优先保留并检查。

避免首次初始化中途结束后，下次重新运行破坏已有成果。

# 13. 大文件读取原则

长期运行后：

- `FEEDBACK_INDEX.md`
- `DECISIONS.md`
- `CONFLICTS.md`

都可能逐渐增大。

恢复时：

> 优先按 ID、来源或关键词搜索需要的条目，而不是默认整份重新加载。

初始化和全局复盘确有必要时例外。

# 14. FB：稳定原始反馈单元

`FB` 表示：

> **一条可以稳定定位的原始反馈表达。**

例如：

- 一个 Issue；
- 一条反馈记录；
- 一行表格；
- 一个 bullet；
- 一张独立截图反馈；
- 一段连续口语反馈。

FB 不是需求，也不是需求组。

允许：

```
一个 FB → 多个 REQ
多个 FB → 一个 REQ
```

后续需求理解改变：

> 不拆、不合、不重编号原 FB。

需求归并发生在 REQ 层。

# 15. FEEDBACK_INDEX.md

负责：

- FB ID；
- 原始来源；
- source locator；
- 简短内容；
- 反馈级处理状态；
- 重复关系；
- FB 间冲突关系；
- 疑似过期 / 已过期；
- 是否已经进入需求分析。

FB 层状态可以包括：

- 已索引；
- 重复；
- 冲突；
- 疑似过期；
- 已过期；
- 待代码核实；
- 已代码核实；
- 已进入需求。

不得包含：

- `planning-ready`
- `待复核`

这两项属于 REQ。

# 16. 渐进式反馈索引

首次运行不要求一次深读全部反馈材料。

先做：

> **浅层盘点**

识别：

- 有哪些文件；
- 有哪些目录；
- 哪些来源已经扫描；
- 哪些来源尚未扫描。

然后分批建立 FB。

`FEEDBACK_INDEX.md` 必须能反映反馈来源扫描进度。

如果输出目录处于反馈目录内部，扫描必须排除自身输出。

默认旁级输出目录优先避免此问题。

# 17. FB 冲突与 CON 的边界

两条原始反馈互相矛盾：

> 先只在 `FEEDBACK_INDEX.md` 标记 FB 层冲突。

如果矛盾需要用户选择：

> 进入 Q / DEC。

只有当问题已经上升为：

- 多个 REQ 之间的冲突；
- 全局业务约束；
- 跨模块设计冲突；

才进入 `CONFLICTS.md`。

不是所有 FB 矛盾都创建 CON。

# 18. DECISIONS.md

只保存正式用户 Decision。

使用稳定 ID：

```
DEC-001
DEC-002
```

重要 Decision 不得只存在于聊天。

Decision 采用 append-only。

用户改变决定：

```
DEC-009
supersedes: DEC-003
```

保留旧 Decision，不覆盖。

# 19. CONTEXT.md

仅保存：

> **影响多个需求的共享 Context Fact。**

每项使用：

```
CTX-001
CTX-002
```

应注明必要的：

- 事实正文；
- 来源；
- 适用范围。

只影响单个 REQ 的 Context Fact 留在对应 REQ 中。

# 20. CURRENT_QUESTIONS.md

只保存：

> **当前真正需要用户回答的问题。**

每个当前问题可使用临时 `Q-xxx` 标识。

至少包含：

- 背景；
- 为什么需要确认；
- 选项或需要补充的信息；
- 默认推荐；
- 不同答案的影响。

用户回答后：

### Decision

进入 `DECISIONS.md`。

### Context Fact

进入对应 REQ 或 `CONTEXT.md`。

然后：

> 从 `CURRENT_QUESTIONS.md` 移除。

不维护历史 Q 档案。

# 21. CONFLICTS.md

保存跨 REQ 的冲突与全局约束。

使用：

```
CON-001
CON-002
```

分为两类：

## blocking

会改变需求方向。

未解决时阻塞相关 REQ 达到 `planning-ready`。

## constraint

不会推翻需求方向，但后续规划必须考虑的：

- 依赖；
- 顺序；
- 联动；
- 全局约束。

constraint 可以随 ready REQ 一起交给规划阶段。

# 22. REQ 一项一文件

每个正式需求独立：

```
requirements/REQ-007.md
```

避免大型需求主文档随着项目不断膨胀。

恢复时只读取当前相关的少量 REQ。

# 23. REQ 稳定 ID

REQ 建立后：

> ID 不复用，不改变原有语义。

如果需要拆分：

- 创建新 REQ；
- 原 REQ 使用 `superseded_by` 指向新需求。

如果需要合并：

- 保留一个 canonical REQ；
- 其余 REQ 使用 `merged_into` 指向 canonical REQ。

旧 ID 不删除。

# 24. REQ 最小字段

每个 REQ 至少包含：

- ID；
- 标题；
- `source_fb`；
- 用户实际问题；
- 目标结果；
- 当前关键 Technical Fact；
- 必要 Context Fact / `CTX` 引用；
- 推荐方向；
- 当前未决问题；
- `touches`；
- readiness。

按需求实际情况增加：

- 备选方案；
- UI 影响；
- 数据影响；
- 权限影响；
- 状态流影响；
- 风险；
- 回归点；
- 测试关注点。

## 24.1 条件必填引用

如果 REQ 实际依赖：

- DEC；
- CTX；
- CON；
- 后续阶段再确认事项；

则必须显式记录相应：

- `DEC-xxx`
- `CTX-xxx`
- `CON-xxx`
- 触发点

不能只在正文中模糊描述。

尤其 `planning-ready` REQ 必须保留其关键依据引用。

# 25. Technical Fact 的代码引用

重要 Technical Fact 记录：

> **简短事实 + 文件路径 + 符号 / 组件 / 接口名称**

不依赖：

- Git commit；
- 行号；
- branch。

这样既方便重新定位，也不引入版本维护机制。

# 26. Single Source of Truth

同一内容只维护一个权威正文。

- 原始反馈 → `FEEDBACK_INDEX.md`
- 正式需求 → `REQ-xxx.md`
- 用户 Decision → `DECISIONS.md`
- 跨 REQ Context Fact → `CONTEXT.md`
- 跨 REQ 冲突 → `CONFLICTS.md`
- 当前问题 → `CURRENT_QUESTIONS.md`
- 当前恢复状态 → `ENTRY.md`

ENTRY 只引用。

FB 与 REQ 的正式映射只维护：

> ```
> REQ.source_fb
> ```

需要从 FB 反查 REQ 时按 FB ID 搜索。

# 27. touches

`touches` 是有限上下文下发现跨需求关系的轻量索引。

允许按项目实际存在的信息填写，例如：

```
touches:
  modules:
    - auth
  entities:
    - User
  permissions:
    - document.read
  state_flows:
    - ingestion
  flows:
    - document-upload
  contracts:
    - DocumentCreateRequest
  surfaces:
    - web-admin
```

全部字段均按需使用。

## 27.1 touches 命名规则

优先使用项目真实 identifier：

- module / package 名；
- entity / table 名；
- permission key；
- 状态或状态流名称；
- 已有用户流程名；
- API / 数据契约名；
- 实际端或页面区域名。

新增 touches 前：

> 先搜索已有 REQ 的 touches，已有同义 identifier 时优先复用。

不要随意使用：

> “权限相关逻辑”“文档这块功能”

这类无法稳定匹配的自由描述。

不额外建立复杂 taxonomy。

## 27.2 touches 的作用

完成 REQ 细化后：

1. 搜索其他 REQ 的 touches；
2. 命中相同或明显相关 identifier 时；
3. 只读取少量命中的 REQ；
4. 判断是否触发全局复盘。

`touches` 只是辅助索引。

AI 已明显发现多个需求相关时，即使字符串没有命中，也可以触发复盘。

# 28. 七步主工作流

## Step 1：恢复或初始化

解析输入、输出和代码位置。

存在 ENTRY：

> 恢复。

不存在 ENTRY：

> 幂等初始化。

### Completion Criteria

已明确：

- 输入位置；
- 输出位置；
- 代码根目录或合理路径假设；
- ENTRY；
- 反馈索引；
- 当前工作单元。

## Step 2：理解当前反馈单元

处理一个反馈主题或一组高度相关 FB。

明确：

- 来源；
- 重复关系；
- FB 冲突；
- 疑似过期项；
- 当前需要验证的核心问题。

### Completion Criteria

当前 FB：

- 有稳定 ID；
- 来源可追溯；
- 已知重复明确；
- 已知冲突记录；
- 需要代码验证的问题明确。

## Step 3：核实 Technical Fact

带着具体问题读取当前代码。

优先检查当前问题实际涉及的：

- 页面 / 组件；
- 路由；
- API；
- Service；
- 数据模型；
- 权限；
- 状态流；
- 后台任务；
- 配置；
- 测试；
- 数据契约；
- 上下游依赖。

不要无目的扫描整个代码库。

### Completion Criteria

所有影响需求判断的重要技术陈述：

- 已有项目证据支持；

或者：

- 明确标记为尚未核实 / 证据不足。

## Step 4：需求细化

把反馈转化为下一版本需求判断。

明确：

- 用户实际问题；
- 目标结果；
- 当前实现与目标之间的差距；
- 推荐方向；
- 合理备选方向；
- 主要影响范围；
- 风险与回归点；
- 当前未决问题。

不得凭空扩展没有反馈、场景或证据支持的需求。

### Completion Criteria

至少已经明确：

- 问题；
- 目标；
- 当前差距；
- 推荐方向；
- 主要影响；
- 主要风险；
- 是否存在 Decision / Context Fact 缺口。

完成后更新 touches 并执行关联 REQ 搜索。

## Step 5：处理必要用户问题

先判断未知属于：

### Technical Fact

继续查项目证据。

### Context Fact

必要时向用户确认现实业务背景。

### Decision

进入 Decision Frontier。

# 29. Decision Frontier

Decision 可能存在依赖。

如果一个问题依赖另一个尚未解决的问题：

> 不同时询问。

当前只向用户询问：

> **前置条件已解决、当前可以独立回答的问题。**

不建立复杂决策图。

每轮只询问少量真正会改变需求方向的问题。

# 30. 追问原则

优先询问会改变：

- 需求方向；
- 范围；
- 业务规则；
- 默认行为；
- 关键交互；
- 数据模型；
- 权限；
- 开发范围

的问题。

不要询问可以自己核实的 Technical Fact。

用户长期未回答 blocking Decision：

> REQ 保持待用户确认。

不能把 AI 推荐自动升级成用户 Decision。

## Step 6：全局复盘

以下情况应考虑全局复盘：

- 多个 REQ 触及同一模块；
- 多个 REQ 影响同一用户流程；
- 同一数据结构或数据契约被多处影响；
- 权限、状态流或端之间发生交叉；
- 局部推荐互相冲突；
- 一个 Decision 影响多个需求；
- 一个局部需求会破坏整体一致性。

### Completion Criteria

已经发现跨项影响时：

- 完成必要记录和处理；

或者：

- 明确确认仍可独立处理。

## Step 7：收束

以下情况均可以结束当前轮：

- 当前工作单元完成；
- 已形成稳定需求；
- 达到 planning-ready；
- 需要用户回答；
- 发现需要下一轮处理的全局问题；
- 当前上下文已经积累足够信息；
- 下一工作单元已经明确。

结束前完成文档维护并最后更新 ENTRY。

# 31. 文档写入顺序

当一轮需要修改多个文档时，采用固定维护顺序：

```
DECISIONS / CONTEXT
→ CONFLICTS
→ REQ
→ FEEDBACK_INDEX
→ CURRENT_QUESTIONS
→ ENTRY
```

不存在的步骤跳过。

`ENTRY.md` 永远最后更新。

这是：

> **简单的文档维护纪律**

不是事务、一致性协议或锁机制。

恢复时如发现明显残留矛盾，根据各文档权威职责完成清理即可。

# 32. planning-ready

`planning-ready` 表示：

> **该需求已经足够成熟，可以交给下一版本迭代规划阶段。**

不表示：

- 技术方案已设计完成；
- 开发任务已拆分；
- implementation-ready；
- 所有未知完全消失。

# 33. planning-ready 条件

REQ 原则上必须满足：

1. 用户问题明确；
2. 目标结果明确；
3. 关键 Technical Fact 已核实；
4. 必要 Context Fact 已明确；
5. 推荐方向明确；
6. 所有会改变需求方向的 Decision 已解决；
7. 主要影响范围已知；
8. 不存在未解决 blocking CON；
9. 主要风险、回归点和测试关注点已识别；
10. 剩余未知不阻塞规划。

如果仍存在非阻塞未知：

每一项必须注明：

- 为什么当前不阻塞；
- 什么节点需要再次确认；
- 具体触发条件；
- 不确认可能产生什么风险。

没有明确后续触发点的未决问题：

> 不得进入 planning-ready。

# 34. planning-ready 证据指针

ready 不能只是模型自评。

REQ 应能够通过 ID 或代码引用指向其关键依据：

- source FB；
- Technical Fact / code reference；
- CTX；
- DEC；
- CON；
- 后续触发点。

不建立额外 readiness matrix。

# 35. planning-ready 的失效

不进行任何后台监控或 Git 检测。

如果一个已经 ready 的 REQ 发生**实质修改**：

> 修改前先将 readiness 调整为 `待复核`。

实质修改包括会改变其结论的：

- 用户问题；
- 目标结果；
- 关键 Technical Fact；
- 关键 Context Fact；
- 推荐方向；
- 关键 DEC；
- blocking CON；
- 全局复盘结论。

以下通常不属于实质修改：

- 措辞调整；
- 格式调整；
- 引用路径修正；
- 不改变范围和方向的 touches 整理。

典型失效情况：

1. 依赖的 DEC 被新 DEC supersede；
2. 引用的 CTX 被用户更正；
3. 新 blocking CON 影响该 REQ；
4. 全局复盘改变其推荐方向；
5. 用户明确说明输入已实质更新并影响该 REQ。

完成重新核实并重新满足 ready 条件后，才能恢复 `planning-ready`。

# 36. “已过期”与“已修复”

必须区分。

## 已过期

反馈描述的需求或业务场景已经不再适用。

## 已修复

用户本次提供的代码快照中已经不存在该问题。

当前代码已经修复：

> 不能自动推断用户真实运行环境已经获得修复。

是否已部署属于 Context Fact。

# 37. 当前迭代质量原则

当前已经能够确定的问题：

> 不应仅因为未来还能继续迭代，就无理由降低方案质量或延期。

同时：

> 不得为了所谓最终形态，主动扩展没有真实反馈、业务目标或代码依据支持的需求。

避免：

- “以后再说”式降质；
- 无依据的过度设计。

# 38. 整体完成条件

Requirement Discovery 阶段可以收束时，应满足：

- 本次约定范围内反馈来源已经完成索引；
- 重复关系已经处理；
- 明显过期反馈有依据；
- 有效 FB 均有明确去向；
- 对应需求已经建立，或明确说明暂时不能建立的原因；
- planning-ready REQ 已明确；
- 重要未决 Decision 没有被隐藏；
- 用户正式 Decision 已持久化；
- 必要 Context Fact 已记录；
- 跨需求 CON 已记录；
- 后续阶段再确认事项拥有明确触发点；
- ENTRY 能够清楚说明最终状态和下一步。

# 39. 下游接口

后续迭代规划阶段主要消费：

1. planning-ready REQ；
2. 对应 DEC；
3. 必要 CTX；
4. 当前 Technical Fact；
5. constraint CON；
6. 带明确触发点的后续待确认事项。

后续规划不应再次从全部原始反馈重新推导需求。

# 40. 推荐 Skill 包结构

```
ruan-requirement-discovery/
├── SKILL.md
└── references/
    ├── feedback-processing.md
    ├── code-verification.md
    ├── requirement-refinement.md
    ├── decisions-and-questioning.md
    └── global-review.md
```

Skills Creator 可以轻微调整文件名，但不要重新改变职责。

# 41. Step → Reference 加载映射

| 当前阶段             | 按需加载                                        |
| -------------------- | ----------------------------------------------- |
| Step 1 恢复 / 初始化 | 主 SKILL 已包含必要规则，通常无需额外 Reference |
| Step 2 反馈处理      | `feedback-processing.md`                        |
| Step 3 代码核实      | `code-verification.md`                          |
| Step 4 需求细化      | `requirement-refinement.md`                     |
| Step 5 用户追问      | `decisions-and-questioning.md`                  |
| Step 6 全局复盘      | `global-review.md`                              |
| Step 7 收束          | 主 SKILL 已包含必要规则                         |

只有真正进入对应步骤时加载相关 Reference。

不要每轮默认读取全部 References。

# 42. Reference 职责

## feedback-processing.md

负责“怎么做”：

- 浅层盘点；
- FB 提取；
- 重复处理；
- FB 冲突；
- 截图；
- 口语反馈；
- 过期判断；
- 渐进索引格式。

## code-verification.md

负责：

- 如何搜索代码；
- 如何形成 Technical Fact；
- code reference 格式；
- Technical / Context Fact 判别；
- 输入更新后的受影响范围核实方法。

## requirement-refinement.md

负责：

- REQ 最小模板；
- 必填 / 条件必填字段；
- touches 格式；
- 影响分析；
- 风险、回归、测试；
- readiness 证据记录格式。

## decisions-and-questioning.md

负责：

- Context Fact 与 Decision 的具体判别；
- Decision Frontier 的具体处理；
- CURRENT_QUESTIONS 格式；
- DEC / CTX 记录格式；
- `supersedes` 使用方式。

## global-review.md

负责：

- touches 命中后的复盘方式；
- CON 格式；
- blocking / constraint 判别；
- 跨需求冲突处理；
- 全局设计判断格式。

# 43. 最小模板

Skills Creator 实现时应提供稳定的最小格式，而不是让每轮自由发明。

至少固定：

- `ENTRY.md`
- `REQ-xxx.md`
- DEC 条目
- CTX 条目
- CON 条目
- `CURRENT_QUESTIONS.md`

模板只固定恢复与引用所需字段。

不要把模板设计成长篇 PRD。

# 44. Skills Creator 实现要求

正式生成 Demo 时必须保持：

### 1. 主 SKILL 足够薄

如果实现后 SKILL.md 明显膨胀，应优先检查是否把 Reference 内容重复写进主文件。

### 2. Completion Criteria 不得删除

这是防止 Agent 过早跳步的核心机制。

### 3. Stable Input Contract 不得被实现成版本系统

不得自行加入：

- Git commit；
- branch 基线；
- diff；
- 自动文件变化检测；
- 并发协调。

### 4. 三类信息边界必须保留

- Technical Fact
- Context Fact
- Decision

### 5. Single Source of Truth 必须保留

不要为了方便恢复复制正文。

### 6. Decision Frontier 必须保留

但不要构建复杂决策图系统。

### 7. `touches` 必须轻量

不建立新的项目 taxonomy 或领域模型。

### 8. planning-ready 必须保留

这是和下游迭代规划的正式接口。

### 9. 固定运行文档必须保留

不同会话不要重新发明目录和文件职责。

### 10. 保持文档驱动

不得增加：

- 数据库；
- JSON 状态机；
- 锁；
- 自动任务协调；
- 隐式同步；
- 复杂版本一致性机制。

# 45. 试跑重点

设计冻结后，不再继续进行纯纸面扩展。

Skill Demo 完成后，应使用真实反馈目录试跑若干轮，重点观察：

### 1. 跨会话恢复

新会话是否能从 ENTRY 正确恢复，而不会创建第二套任务。

### 2. touches 一致性

不同轮次是否能够复用项目真实 identifier，避免同一对象出现多个同义名称导致关联失效。

### 3. planning-ready 证据

ready REQ 是否真的保留了必要的：

- FB；
- Technical Fact；
- CTX；
- DEC；
- CON；
- 后续触发点。

### 4. 全局复盘

多个相关 REQ 出现后，touches 是否真的能触发必要的跨项检查。

### 5. 主 Skill 大小

Skills Creator 是否真正实现了：

> **薄 SKILL.md + 按需 Reference**

而不是只在设计上声明“薄”。

# 46. 最终设计原则

`ruan-requirement-discovery 2.0` 最终是：

> **一个长期、可恢复、基于稳定输入工作的需求发现 Skill。**

它使用：

> **固定恢复入口 + 渐进反馈索引 + Technical Fact 核实 + Context Fact + 用户 Decision + 单文件 REQ + 轻量 touches + 全局复盘**

把：

> **真实反馈 + 当前代码 + 真实业务背景**

转化为：

> **planning-ready 需求**

核心原则：

### Stable Input

输入稳定性由明确使用约定保证，不由 Skill 内部版本机制保证。

### Technical facts are discovered

可以从项目证据获得的事实由 AI 自己核实。

### Context facts are supplied when necessary

代码无法知道的真实业务背景，由可靠材料或用户补充。

### Decisions are elicited

真正改变下一版本方向的选择由用户决定。

### Single Source of Truth

每类信息只有一个权威正文位置。

### Progressive Disclosure

主 Skill 只保存控制流和过关条件，具体方法按需读取 Reference。

### Every step has completion criteria

避免事实未核实、问题未明确就继续推进。

### Decision Frontier

只询问当前真正能够独立回答并会影响方向的问题。

### planning-ready

明确规定什么时候需求足够成熟，可以交接给后续迭代规划。

最终目标不是构建复杂的 Requirements Management System，而是：

> **通过尽可能轻的机制与明确的使用约定，让 AI 能够长期、可靠、可恢复地把真实反馈和真实项目事实整理成下一版本真正可规划的需求。**