# 追问、Decision 和 Context Fact

## 先判别未知类型

- 能从当前项目快照、配置、测试或接口查明：继续核实 Technical Fact。
- 描述快照之外的现实环境：必要时询问 Context Fact。
- 存在多个合理方向且选择会改变版本目标、范围、规则、默认行为或关键交互：进入 Decision Frontier。

不要把技术事实校验交给用户，也不要同时询问依赖尚未解决的后置问题。每轮只保留少量当前可以独立回答的问题。

## CURRENT_QUESTIONS.md

当前问题可使用临时 `Q-xxx`，每个问题至少包含：

```markdown
### Q-001
- background: 来源 FB、代码事实和当前缺口
- why_confirmation: 不确认会改变什么判断
- options: 选项或需要补充的信息
- default_recommendation: 基于现有事实的推荐
- impact: 影响的 REQ、流程、模块、数据、权限或端
```

用户回答后，若形成正式取舍，追加 DEC；若形成共享现实背景，追加 CTX。DEC/CTX 至少保留：

```markdown
### DEC-001
- source_q: Q-001
- question_summary: 问题摘要
- user_decision: 用户回答的准确转述
- supersedes: null
- affected_requirements: [REQ-001]
```

`CURRENT_QUESTIONS.md` 只保留当前未回答问题，已回答问题移除；上述最小追溯信息留在 DEC/CTX 中。用户改变决定时新增 DEC 并写 `supersedes`，不要覆盖旧记录。共享 CTX 被纠正时采用同样的轻量修订关系。

## 默认推荐

推荐选项必须说明依据和影响。用户未确认时，REQ 保持 `待用户确认`；不能因为推荐看起来合理就升级为 `planning-ready`。
