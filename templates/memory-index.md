# {{product_name}} Memory Index — 最近 20 条新增

> 索引格式：`- [<file-stem>](<cat>/<file>.md) — Phase-N` （每行 ≤ 80 字符）
> 全量索引见 `memory/.index.md`（按 category 分组，懒读）。
> Phase 9 任务的 Phase A 强制读：本文件 + 关键词命中文件。

## 最近新增（FIFO 滚动窗口，最多 20 条）

_暂无 — 第一个 memory 写入后开始填充_

## 写入触发条件

| 类别 | 触发 |
|------|------|
| decision | ≥ 2 方案非显而易见取舍（如选 React vs Vue 且各有强约束） |
| pitfall | 第三方/系统非显而易见行为（如某 API 限流、某依赖坑） |
| convention | 跨任务隐性约定（如所有日期都用 UTC ISO） |
| integration | 外部系统版本/协议细节（如某接口字段类型不符文档） |

## 推翻规则

新决策与旧决策冲突时：
1. 旧文件状态字段标 `superseded-by-<新文件名>`
2. 索引行末追加 `(已废弃)`
3. **不删除**旧文件（保留历史推翻轨迹）
