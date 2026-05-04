# Stage 08 — Story Breakdown（任务拆分）

> **本阶段职责**：把 PRD 的 P0 功能拆成单会话可完成、可独立验证的 stories；初始化 `08-stories/PROGRESS.md` 任务状态板。产物是 `08-stories/PROGRESS.md` + 多个 `08-stories/T-x.x.x.md`。
>
> **核心理念**：好的 task 满足 4 条 — 单会话能做完、可独立验证、依赖明确、粒度均匀（API 端点 / DB 迁移 / 前端组件 / 队列+Processor 各 = 1 任务）。

---

## 入口检查

1. **必须**：检查 `00-state.md` 是否有 `gate-passed: <date>` 标记
   - 没有 → 立即停止，输出："Discovery Gate 未通过。请先回 Phase 7 末尾确认进入实现，回复'开始实现'。"
2. Read `05-prd.md` §V1 功能清单（P0）
3. Read `07-architecture.md` §推荐栈 + §数据模型
4. Read `06-design.md` §关键 UX 流程

---

## 步骤 1：开场 + Plan Mode 询问

**首先 AskUserQuestion** 问 Plan Mode（如主 SKILL.md 进入路由时还没问）：

```
即将开始 Phase 8：把 PRD 拆成可执行的 stories。

问题：要切到 Claude Code Plan Mode 做这次拆分吗？
（Plan Mode 能让规划更结构化，且 skill 不会立即修改文件，方便你审阅）

- 是，进入 Plan Mode（推荐 — 拆分是关键决策点）
- 否，直接拆
```

如选"是"：提示用户按 `Shift+Tab`，等待用户确认在 Plan Mode 后继续。
如选"否"：在 `MEMORY.md` 写 1 条 `decision` 类 memory（"Phase 8 拆分时主动放弃 Plan Mode"）后继续。

---

## 步骤 2：项目初始化（一次性，仅在第一次进 Phase 8 时执行）

### 2.1 决定项目代码目录位置

AskUserQuestion：

```
代码放在哪里？
- 当前 product/{{slug}}/ 内（默认，规划文档和代码同目录，方便 git 管理）
- 当前工作目录的根（product/ 文档与 src/ 代码并列）
- 自定义路径（你提供）
```

记录到 `00-state.md` 的 `## profile` 节末追加 `code-root: <path>`。

### 2.2 初始化代码项目

根据 Phase 7 推荐栈生成初始化命令，AskUserQuestion 让用户确认是否执行：

```
即将执行以下命令初始化项目骨架：

  cd <code-root>
  {{初始化命令，如 pnpm create next-app@latest . --typescript --tailwind --app}}

要执行吗？
- 是，执行
- 否，我自己来（你先把骨架建好再回来）
- 让我看完整命令再决定
```

**严格规则**：
- 只在用户明确"是"后才执行 Bash
- 优先用 `pnpm` 而非 `npm`/`yarn`（用户全局偏好）
- 安装命令用 `--yes` / non-interactive 标志
- 完成后用 `ls` 验证目录结构

### 2.3 复制 PROGRESS / story 模板

```
mkdir -p <product/slug>/08-stories
```

读 `~/.claude/skills/dream-weaving/templates/progress.md` → 替换占位符 → Write 到 `<product/slug>/08-stories/PROGRESS.md`

---

## 步骤 3：拆分原则（必读）

**好任务 4 条**：
1. **单会话可完成** — 一次 Phase 9 微循环（A→B→C→D）能跑完
2. **可独立验证** — 不依赖其他未完成任务就能跑
3. **依赖明确** — 显式列出前置 T-x.x.x
4. **粒度均匀** — 每个任务约 30-90 分钟工作量

**任务粒度对照表**：

| 类型 | 1 个任务的范围 |
|------|-------------|
| 数据库迁移 | 1 张表 + 该表的索引 |
| API 端点 | 1 个 endpoint + 入参校验 + 错误处理 |
| 前端组件 | 1 个完整可用组件 + 状态（含 loading / error / empty）|
| 前端页面 | 1 个 route + 数据获取 + 1-2 个组件组合 |
| 后端逻辑 | 1 个独立函数 / 1 个队列 processor |
| 集成 | 1 个第三方服务的接入（Auth / 支付 / OSS）|
| 配置 | 1 类配置（CI / 部署 / 环境变量）|

**任务编号规则**：`T-{Sprint}.{Module}.{Seq}`
- Sprint：当前迭代号，V1 一般 = 1
- Module：模块号，按 PRD 的 P0 功能 F-1, F-2... 对应
- Seq：模块内顺序，从 1 开始

例：`T-1.1.1` = Sprint 1 / Module 1（PRD F-1）/ 第 1 个任务

---

## 步骤 4：依次拆每个 P0 功能

对 PRD §V1 P0 列表的每个 F-N，按以下顺序拆：

### 标准拆分套路（按依赖顺序）

1. **数据层**（如有新表）
   - `T-1.N.1` 数据模型 / 迁移
2. **后端**（如有 API）
   - `T-1.N.2` API endpoint 实现
   - `T-1.N.3` 业务逻辑（如复杂）
3. **前端**
   - `T-1.N.4` UI 组件
   - `T-1.N.5` 页面集成
4. **测试 / 联调**
   - `T-1.N.6` 端到端验证（按 PRD 验收标准）

**实际拆分时**：根据 F-N 的具体形态调整。简单功能可能合并到 2-3 个任务；复杂功能可能展开到 6-8 个任务。

### Plan Mode 触发（每个 story 拆完后判断）

如某个 task 命中以下任一条件，**AskUserQuestion 询问**是否进 Plan Mode 实施它：

- 关键文件 ≥ 5 个
- 涉及数据迁移
- 涉及不可逆操作（如删除既有数据、外发消息、调用付费 API）
- 跨模块依赖 ≥ 3 个

询问模板：

> Story T-x.x.x 涉及 [具体原因]，建议在 Phase 9 实施时进入 Claude Code Plan Mode 做规划。我先在 story 文件里标记一下，等到那时提醒你。

确认后在该 `T-x.x.x.md` 的 §Plan Mode 建议节标 ✅ 推荐。

---

## 步骤 5：写每个 story 文件

对每个 task：
1. 读 `~/.claude/skills/dream-weaving/templates/story.md` 模板
2. 替换占位符
3. 写到 `<product/slug>/08-stories/T-x.x.x.md`
4. 同时把简化版加入 `PROGRESS.md` 的任务列表

**story 文件必填项**（从模板继承）：
- 用户故事（WHY/WHAT）
- 验收标准（Given/When/Then）
- 边界条件
- 文件变更计划（具体路径）
- 关键实现要点
- 验证策略（自动化 vs 手测预判）
- Plan Mode 建议（如命中触发条件）

---

## 步骤 6：更新 PROGRESS.md

格式：

```markdown
## 当前 Sprint

**Sprint ID**: S-1
**目标**: V1 实现 PRD §V1 Single Most Important Workflow
**起始日期**: {{date}}
**任务总数**: {{N}}
**已完成**: 0/{{N}}

## 任务列表

- [ ] **T-1.1.1** 数据库 schema：新建 records 表
  - 前置依赖: 无
  - 关键文件: `db/migrations/0001_init.sql`
  - 验证方式: 跑 migration 后 \d records 显示字段齐全

- [ ] **T-1.1.2** API: POST /api/records
  - 前置依赖: T-1.1.1
  - 关键文件: `app/api/records/route.ts`
  - 验证方式: curl 创建一条记录返回 201

...
```

---

## 步骤 7：回放 + 校对

```
✅ Phase 8 任务拆分完成

Sprint S-1 共 {{N}} 个任务：
  - F-1 ({{F-1 名}}): T-1.1.1 ~ T-1.1.{{x}}（{{N1}} 个任务）
  - F-2 ({{F-2 名}}): T-1.2.1 ~ T-1.2.{{y}}（{{N2}} 个任务）
  - ...

预估总工时：{{N × 60}} 分钟（约 {{H}} 小时）
建议进 Plan Mode 的复杂任务：{{M}} 个

接下来：
- 通读 product/{{slug}}/08-stories/PROGRESS.md 和各 T-x.x.x.md
- 准备好后用 /dream-weaving implement 进入 Phase 9 开始第一个任务
- 每完成一个任务，会用 Phase A→B→C→D 微循环（详见 Phase 9）
```

---

## 步骤 8：Exit gate

- [x] `08-stories/PROGRESS.md` 存在且至少 1 条任务
- [x] 每条任务对应一个 `T-x.x.x.md` 文件存在
- [x] 每个 story 文件含验收标准 + 文件变更计划 + 验证策略
- [x] `00-state.md` 阶段进度勾选 Phase 8

满足后等用户运行 `/dream-weaving implement` 进入 Phase 9。

---

## 反例

- ❌ 没有 Discovery Gate 标记就开始拆
- ❌ 把 1 个任务拆成 30 分钟以下（过细）或 4 小时以上（过粗）
- ❌ 任务依赖循环（A 依赖 B，B 又依赖 A）
- ❌ 验收标准写"功能正常"
- ❌ 不区分自动化 vs 手测，全留空让 Phase 9 现想
- ❌ 跳过 Plan Mode 询问（即使简单任务也至少在拆分阶段问一次）
