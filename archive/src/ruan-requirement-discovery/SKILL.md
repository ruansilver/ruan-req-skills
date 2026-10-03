---
name: ruan-requirement-discovery
description: 面向软件项目下一版本的需求发现与需求分析。只要用户提供用户反馈目录、问题记录、截图、口语反馈或半成品需求，并要求结合当前代码整理需求、核实现状、追问取舍、处理冲突或产出可规划需求，就使用本 Skill。它维护可恢复的 Markdown 需求文档，不实现需求、不修改业务代码，也不替代后续迭代规划。
---

# Ruan Requirement Discovery

把真实反馈、当前项目快照和必要的业务背景，逐步整理为可追溯、可恢复、可交给下一版本规划阶段消费的需求。保持轻量：用明确的使用约定、稳定 ID、固定文档职责和少量 sanity check 解决问题，不构建版本系统、数据库、锁、状态机或后台协调器。

## 何时使用

当用户给出反馈目录，或要求分析下一版本需求、核实反馈与代码的关系、合并重复反馈、处理冲突、向用户追问业务取舍、维护长期需求分析状态时使用。可以读取和分析代码、配置、测试、接口、数据结构和 UI，但不修改业务代码，不执行正式开发，不替用户自动决定产品方向。

## 输入约定

- 用户反馈目录是必需输入。材料可以重复、冲突、过期、不完整，或只有截图和口语记录。
- 需求收集目录可由用户指定；未指定时，在反馈目录旁创建 `<反馈目录名>-requirement-discovery/`。
- 当前代码库用于核实现状。代码根目录不明确时，在 `ENTRY.md` 记录路径假设。
- 一次工作持续期间，默认用户不同时修改分析所依据的代码和原始反馈。用户需要修改时，应提供副本或固定快照，并明确告知“输入已更新”。
- Skill 不判断 Git branch、commit、diff、hash、版本同步或并发修改。可记录用户提供的 `snapshot_label` / `input_note` 供人理解，但不参与自动一致性判断。

## 固定输出

在需求收集目录维护以下结构：

```text
ENTRY.md
FEEDBACK_INDEX.md
CONTEXT.md
DECISIONS.md
CONFLICTS.md
CURRENT_QUESTIONS.md
requirements/
  REQ-001.md
  REQ-002.md
```

这些文件的职责是单一权威正文：原始反馈在 `FEEDBACK_INDEX.md`，正式需求在 REQ 文件，用户决定在 `DECISIONS.md`，共享 Context Fact 在 `CONTEXT.md`，跨需求冲突在 `CONFLICTS.md`，当前问题在 `CURRENT_QUESTIONS.md`，恢复状态和指针在 `ENTRY.md`。不要为了恢复而复制正文。

## 三类信息

### Technical Fact

由项目快照内部证据发现：代码、配置、测试、API、数据结构、权限、状态流、后台任务和 UI 实现。记录简短事实、路径和符号/组件/接口名称。没有证据的技术判断标记为未核实。

### Context Fact

项目快照之外的现实环境：生产环境行为、真实部署情况、现场使用方式、用户角色、业务背景和外部系统约束。只影响一个 REQ 的事实留在 REQ；影响多个 REQ 的共享事实写入 `CONTEXT.md`，分配稳定 `CTX-xxx`。不要把代码推断伪装成 Context Fact。

### Decision

多个方向都与现有证据相容，且选择会改变下一版本方向、范围、业务规则、默认行为或关键交互时，向用户询问并记录正式 Decision。AI 推荐不是用户 Decision；用户明确选择或确认后才写入 `DECISIONS.md`。

## 恢复与初始化

每轮先定位输出目录并读取 `ENTRY.md`。有 `ENTRY.md` 就按其中的当前工作单元和必要指针恢复；没有就做幂等初始化：不覆盖已有非空文件，只创建缺失文件，并建立浅层反馈索引和初始入口。

`ENTRY.md` 至少维护：任务目标与边界、反馈目录、输出目录、代码根目录、`analysis_scope`、当前工作单元、已处理/处理中/待确认范围、当前阻塞、当前问题路径、相关 CON、下一步和必要文档指针。`analysis_scope` 要写清纳入的目录/文件/来源、排除项和完成条件。完成条件是：范围内反馈来源已按约定扫描，进入处理范围的 FB 都有 disposition，没有未处理且未说明原因的有效 FB。

## 七步主流程

### Step 1：恢复或初始化

明确输入、输出、代码路径和分析范围，建立或读取 ENTRY。完成标准：恢复入口有效，当前工作单元明确，反馈来源扫描进度可追踪。

### Step 2：理解反馈单元

每轮处理一个主题或一组高度相关 FB。为每条原始反馈分配稳定 `FB-xxx`，记录来源和 locator，标记重复、冲突、疑似过期，并明确需要代码核实的核心问题。具体方法见 `references/feedback-processing.md`。

每个进入处理范围的 FB 最终要有 disposition：`requirement`、`duplicate`、`expired`、`context-only`、`decision-only`、`out-of-scope` 或 `deferred`。`deferred` 必须写原因和后续触发条件。必要时写 `disposition_reason`。

### Step 3：核实 Technical Fact

带着具体问题读取相关页面、组件、路由、API、服务、数据模型、权限、状态流、配置、测试和上下游依赖。避免无目的扫描整个代码库。完成标准：影响需求判断的重要技术陈述有项目证据，或明确标为证据不足。具体方法见 `references/code-verification.md`。

### Step 4：细化 REQ

把反馈转化为下一版本需求判断：用户实际问题、目标结果、当前差距、目标用户/角色、当前范围、明确非目标、推荐方向、主要影响、风险、回归点、测试关注点和 success signal。success signal 只描述改完后可观察的目标现象，不代替开发验收体系。完成后更新 `touches` 并搜索相关 REQ。具体模板见 `references/requirement-refinement.md`。

### Step 5：处理必要追问

先判断缺口属于 Technical Fact、Context Fact 还是 Decision。能从项目证据查到的继续查；只有现实背景和产品取舍无法推断时才追问。每轮只询问前置条件已解决、会改变方向且当前可以独立回答的问题。更新 `CURRENT_QUESTIONS.md` 后再向用户提问。

用户回答后，把正式选择写入 DEC，把共享事实写入 CTX；相关条目保留 `source_q`、问题摘要和用户回答的准确转述，然后从当前问题文件移除。具体规则见 `references/decisions-and-questioning.md`。

### Step 6：全局复盘

多个 REQ 触及同一模块、流程、数据契约、权限、状态流或端，或局部推荐互相冲突时，搜索 `touches` 并只读取命中的少量 REQ。需要跨需求处理时写入 `CONFLICTS.md`：`blocking` 会阻塞相关 REQ 达到 planning-ready；`constraint` 交给下游规划考虑。若确认可独立处理，也在相关 REQ 留下简短结论。具体方法见 `references/global-review.md`。

### Step 7：收束

在当前工作单元完成、形成稳定需求、达到 planning-ready、需要用户回答、发现全局问题或上下文已足够时收束。按以下顺序维护文档，并永远最后更新 ENTRY：

```text
DECISIONS / CONTEXT → CONFLICTS → REQ → FEEDBACK_INDEX → CURRENT_QUESTIONS → ENTRY
```

收束前只对当前工作单元做最小 sanity check：引用的 FB/DEC/CTX/CON 是否存在，是否有明显重复 ID，关系指针是否悬空或自引用，ready REQ 的关键依据是否仍在，ENTRY 指针是否有效。整个范围最终收束时再做一次全局 ID 检查。

## REQ 和 readiness

REQ 一项一文件，ID 不复用。REQ 至少包含：ID、标题、`source_fb`、用户实际问题、目标结果、关键 Technical Fact、必要 Context Fact/CTX、推荐方向、当前未决问题、`touches` 和 readiness。按实际需要增加 UI、数据、权限、状态流、风险、回归、测试、目标用户、范围、非目标和 success signal。

REQ readiness 只使用少量语义：`待分析`、`待用户确认`、`planning-ready`、`待复核`、`withdrawn`。拆分时旧 REQ 用 `superseded_by`，合并时非 canonical REQ 用 `merged_into`。这些是描述标签，不构成完整状态机。

`planning-ready` 表示后续迭代规划不必再次猜用户要解决什么。原则上要求：问题和目标明确；关键 Technical Fact 已核实；必要 Context Fact 已明确；会改变方向的 Decision 已解决；目标用户、范围、非目标和主要影响已知；没有未解决 blocking CON；风险、回归点、测试关注点和 success signal 已识别。剩余未知必须说明为什么不阻塞、何时触发再次确认以及不确认的风险。

## 输入更新

用户明确说“输入已更新”后，先使用用户给出的模块、路径、功能或反馈来源范围，通过 Technical Fact 路径/符号、REQ `touches` 和 `source_fb` 定位受影响 REQ 并重新核实。用户无法说明范围时，优先复核当前 `planning-ready` REQ。只在这次明确通知后处理，不做后台监控。

## 轻量引用规则

重要引用使用稳定 ID：`FB-xxx`、`REQ-xxx`、`DEC-xxx`、`CTX-xxx`、`CON-xxx`、临时 `Q-xxx`。DEC 采用 append-only，用户改变决定时创建新 DEC 并写 `supersedes`。共享 CTX 被纠正时同样可创建新 CTX 并写 `supersedes`；CON 只有被重新定义时才需要 supersede，普通解决记录写 resolution。关键字段和格式按需读取对应 Reference。

## 边界和收束

不要把 AI 推荐写成用户决定，不要把快照外现实写成 Technical Fact，不要因为未来可能继续迭代而降低当前已能确定的方案质量，也不要扩展没有反馈、业务目标或代码依据支持的需求。最终收束时，ENTRY 要能说明范围、每条有效 FB 的去向、planning-ready REQ、未决问题、重要 Decision/CTX/CON 和下一步。Skill 的产物是需求发现与分析文档，不是实现代码或完整 PRD。

## Reference 加载映射

只在进入对应步骤时读取：

| 阶段 | Reference |
| --- | --- |
| Step 2 | `references/feedback-processing.md` |
| Step 3 | `references/code-verification.md` |
| Step 4 | `references/requirement-refinement.md` |
| Step 5 | `references/decisions-and-questioning.md` |
| Step 6 | `references/global-review.md` |

Step 1 和 Step 7 的控制规则已经在本文件中。不要每轮默认读取全部 References。
