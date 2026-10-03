# 全局复盘与 CON

## 触发条件

当多个 REQ 命中相同模块、流程、数据契约、权限、状态流或端，或局部推荐互相冲突时，读取少量相关 REQ，判断共同底层问题和整体影响。先记录关联的 FB/REQ，再决定是否需要 CON。

## CON 分类

```markdown
### CON-001：简短标题
- kind: blocking | constraint
- related_requirements: [REQ-001, REQ-002]
- conflict_or_constraint: 冲突点或全局约束
- candidates: []
- recommendation: 默认方向及原因
- unresolved_questions: []
- resolution: null
```

`blocking` 会阻塞相关 REQ 达到 `planning-ready`；`constraint` 不改变需求方向，但规划阶段必须考虑依赖、顺序、联动或全局约束。问题解决时给 CON 写 `resolution`；只有原 CON 被重新定义或证明错误时才创建新的 CON 并写 `supersedes`。

## 复盘完成标准

发现跨项影响后，要么完成必要记录和处理，要么明确说明相关 REQ 仍可独立处理。不要为了每个相似字符串都创建 CON，也不要用 CON 代替普通 REQ 依赖说明。
