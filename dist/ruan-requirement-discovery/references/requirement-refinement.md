# 需求细化与模板

本 Reference 负责 REQ 字段、`touches`、影响分析和 readiness 依据。

## REQ 最小模板

```markdown
# REQ-001：简短标题

- readiness: 待分析
- source_fb: [FB-001]
- target_users: [相关角色]
- scope: 本次要改变的范围
- non_goals: 明确不包含的范围
- user_problem: 用户实际遇到的问题
- target_outcome: 目标结果
- success_signal: 改完后可观察的目标现象
- technical_facts:
  - 简短事实 + 路径/符号引用
- context_facts: [CTX-xxx 或本 REQ 独有的现实背景]
- recommended_direction: 推荐方向及原因
- alternatives: []
- unresolved_questions: []
- touches:
  modules: []
  entities: []
  permissions: []
  state_flows: []
  flows: []
  contracts: []
  surfaces: []
- impacts: UI / data / permission / state / API / user habit
- risks: []
- regression_points: []
- test_focus: []
- references: DEC / CON / 后续触发点
```

简单需求按实际需要填写；不要机械填充空字段。`source_fb` 是 FB 到 REQ 的正式映射。条件依赖 DEC、CTX、CON 或后续触发点时，显式写稳定 ID。

## touches

优先复用项目真实 identifier：模块/包、实体/表、权限 key、状态流、流程名、接口/数据契约、端或页面区域。新增前先搜索已有 REQ 的 `touches`，避免同一对象出现多个自由描述。`touches` 只是关联索引，不是新建领域 taxonomy。

完成 REQ 后，搜索命中的其他 REQ 并判断是否触发全局复盘。即使字符串未命中，只要已经明显发现跨项影响，也应复盘。若确认可以独立处理，可在 REQ 的 references 或 review note 写明已检查的相关项。

## planning-ready 依据

ready REQ 应能通过 ID 或代码引用回溯：来源 FB、关键 Technical Fact、必要 CTX、已解决 DEC、相关 CON 和非阻塞未知的后续触发点。剩余未知要说明为什么不阻塞、何时重新确认以及不确认的风险。不要把 AI 的默认推荐写成已确认的 Decision。
