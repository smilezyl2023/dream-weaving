# Stage 09 — Implementation（Phase A → B → C → D 微循环）

> **本阶段职责**：执行单个 story（T-x.x.x），用严格的 4 阶段微循环：读上下文 → 实现 → 分层验证 → 三向同步。**不依赖任何外部 skill，完全自包含**。
>
> **核心理念**：每完成 1 个 task 就是 1 次完整 A-B-C-D 循环。**严禁跳步**。

---

## 入口检查

1. Read `<cwd>/product/<slug>/00-state.md`，确认 `gate-passed` 标记
2. Read `08-stories/PROGRESS.md`：找下一个 `[ ]` 任务
3. 如所有任务都 `[✅]` → 提示 "Phase 9 全部完成，运行 /dream-weaving next 进入 Phase 11 用户验证"
4. 如有 `[🔄]` 残留任务（说明上次中断）→ 先警告并询问继续还是回退

---

## 触发动作

参数 `implement` 时执行**一个完整的 A→B→C→D**循环（一个 task）。完成后等用户输入"测试通过"，再触发 Phase D，再提示 `/clear`，再 `/dream-weaving implement` 进入下一个。

---

# Phase A — 读取上下文（任务开始）

## A.1 找下一个任务

1. **Read `08-stories/PROGRESS.md`**：找第一个 `[ ]` 任务
2. **检查状态一致性**：
   - 残留 `[🔄]`：警告 "上次任务 T-x.x.x 状态为进行中，是否继续 / 回退？"
   - PROGRESS 与 story 文件对不上：以 PROGRESS 为准并提示

## A.2 读 story 文件

1. **Read `08-stories/T-x.x.x.md`** 全文
2. 重点关注：
   - 验收标准（决定 Phase C 怎么验证）
   - 文件变更计划（决定 Phase B 改哪些文件）
   - 边界条件（必须实现）
   - Plan Mode 建议（如标 ✅，主动询问用户）
   - 相关 memory（A.4 用）

## A.3 读 MEMORY 滚动窗口

1. **Read `MEMORY.md`**（最近 20 条）：扫标题 + 关键词
2. 不读 `memory/.index.md`（懒读）

## A.4 读 story 声明的 memory

1. story §相关 memory 列出的全部文件 → 全部 Read
2. 这是 spec 作者已主动声明的"必读"memory

## A.5 关键词触发的额外 memory

1. 用 grep 在 `MEMORY.md` 和 `memory/.index.md` 搜：
   - story §文件变更计划 中的路径片段
   - story §关键实现要点 中的技术词（包名、协议、概念）
2. 命中文件**自动 Read**，宁多勿少

## A.6 Plan Mode 二次询问

如 story §Plan Mode 建议 标 ✅ 推荐：

```
🔔 这个 story 在拆分时被标记建议进入 Plan Mode 实施（{{原因}}）。
要切到 Plan Mode 再继续吗？
- 是
- 否（直接实施）
```

如选"是"：提示用户按 `Shift+Tab`，等用户确认后继续。

## A.7 输出上下文摘要

```
📋 任务上下文摘要
当前进度: X/Y · 上一个完成: T-x.x.x · 本次执行: T-x.x.x
关联 story: 08-stories/T-x.x.x.md
相关 memory: [文件列表 或 "无"]
Plan Mode: 已切换 / 未切换
```

---

# Phase B — 实现（按 story 实施代码）

## B.1 标记开始

Edit `PROGRESS.md`：把当前任务的 `[ ]` 改为 `[🔄]`。

## B.2 按 story 实施

1. 严格按 story §文件变更计划 操作：
   - "新增" → Write
   - "修改" → 先 Read 再 Edit
2. 遵循已有项目约定：
   - 包管理器：用户偏好 pnpm
   - 命名：变量英文 + 中文注释（用户偏好）
   - 不写无用注释（用户偏好）
3. 遇到 spec 缺陷（描述与现实不符）：**先更新 spec 再继续**（活文档原则）

## B.3 遇到非显而易见的事 → 写 memory draft

如下场景**暂停实现，先写 memory**：

| 类别 | 触发例子 |
|------|---------|
| decision | 选 React Query vs SWR，发现两者都行但有微妙差异 |
| pitfall | Vercel Postgres 在某条件下慢 10x |
| convention | 这个项目所有日期用 UTC，不用 local |
| integration | 第三方 API v2 字段名和文档不一致 |

写入路径：`memory/<cat>/<topic-slug>.md`

模板：

```markdown
---
status: draft
type: {{decision/pitfall/convention/integration}}
related-tasks: [T-x.x.x]
created: {{date}}
---

## 背景

{{1-2 句什么场景遇到的}}

## 内容

{{决策 / 坑 / 约定 / 集成细节}}

## 影响

{{对未来类似 task 的指导}}
```

写完后回 B.2 继续实现。draft 状态会在 Phase D 改为 active。

## B.4 进度可见性

如本 story 涉及 ≥ 5 个文件，**每完成 2-3 个文件输出 1 行进度**：

```
✅ 已完成 db/migrations/0001.sql
✅ 已完成 lib/db/client.ts
🔄 正在做 app/api/records/route.ts
```

## B.5 严禁

- ❌ 跳过 Phase A 直接编码
- ❌ 改 story 文件以"对齐"实现的偷懒（除非 spec 真的错了）
- ❌ 不写 memory 就把决策"记在心里"
- ❌ 大爆炸提交（一次 20 个文件）

---

# Phase C — 分层验证

> **强制顺序**：C1 → C2 → C3 → C4。**自动化优先，手测兜底，二者都通过才能进 D**。

## Phase C1 — 输出验证清单（需求驱动）

### 规则

- 验证项**必须从 story §验收标准 / §边界条件 推导**
- **禁止反模式**：读完实现再写"测试 X 显示 Y"或"函数入参 A 出参 B"
- **正确做法**：闭眼想"用户拿到这个功能怎么用 / 边界怎么触发 / 错误怎么恢复"，再对照实现
- 6 个维度：主链路 / 副作用 / 边界 / 错误恢复 / 视觉 / 回归
- 每条标 🤖 / 🧑 / 🤖+🧑

### 自动化 vs 手测归类

**判定问题**：「这个 skill（你）能不能在终端跑出 PASS/FAIL？」能跑就 🤖。

| 归类 | 触发条件 | 示例 |
|------|---------|------|
| 🤖 自动化 | shell/node/runner 跑出 PASS/FAIL；无主观；不依赖 OS GUI | API、纯函数、CRUD、tarball grep、`node -e`、文件存在性、`jq` 元数据、headless 表单点击流 |
| 🧑 手测 | 命中下方"手测硬性条件" | 见下表 |
| 🤖+🧑 | 一个验证项需要两者结合 | "API 返回正确" 🤖 + "前端展示符合视觉稿" 🧑 |

**手测硬性条件清单**（命中任一即 🧑）：
1. 视觉 / 动画 / 主观美感
2. 真机硬件（蓝牙 / NFC / 摄像头 / 传感器）
3. 真服务（实际支付 / 真发推送 / 真发邮件）
4. OS 级（系统权限弹窗 / 分享面板 / 通知中心）
5. 主观判断（文案是否得体 / 可访问性体验 / 老人是否看懂）

> "可自动化"≠"必须写 vitest/playwright"。命令行验证（grep / cat / jq / `node -e` / `pnpm test`）也算 🤖。

### C1 输出格式

```markdown
## C1 验证清单 — T-x.x.x

| # | 维度 | 验证项 | 类别 | 来源 |
|---|------|--------|------|------|
| VC-1 | 主链路 | 用户填表单 → 提交 → 看到成功 | 🤖+🧑 | 验收 G/W/T-1 |
| VC-2 | 副作用 | 提交后数据库 records 表 +1 行 | 🤖 | 验收 G/W/T-1 |
| VC-3 | 边界 | 输入超长 → 提示限制 | 🤖 | 边界条件 |
| VC-4 | 错误恢复 | 网络断 → 显示错误 + 可重试 | 🤖 | 边界条件 |
| VC-5 | 视觉 | 按 Phase 6 设计系统 | 🧑 | 设计规范 |
| VC-6 | 回归 | T-1.1.1 创建仍可用 | 🤖 | 已完成任务 |
```

## Phase C2 — 自动化测试（落地 🤖 项）

### 检测项目测试栈

1. 扫 `package.json` / `pyproject.toml` / `go.mod`：
   - playwright / cypress / vitest / jest / pytest / go test
2. 交叉确认 story §验证策略

### 缺工具时

按 story §自动化测试预判 处理：
- 已授权初始化 → Claude `pnpm add -D <tool>` 安装（**仅**用 pnpm）
- 选"跳过自动化" → 全部 🤖 项降级到 🧑
- 未做选择 → AskUserQuestion 三选项后回写 story

### 生成并执行 — 两种落地形式

**A 类（测试文件）**：
- e2e: `tests/e2e/T-x.x.x-<slug>.spec.ts`
- 单元/集成: 跟随项目约定
- **必须**从验收标准推导，不是从实现推导

**B 类（一次性命令验证）**：
- tarball grep / 同源对齐 grep / sandbox `node -e` / 元数据 `jq` / 五连门禁（format/lint/type/test/build）
- 在终端当场跑收集 PASS/FAIL，**不必落 .test.ts**
- **B 类是 🤖，不是 🧑**

### 结果分类

| 结果 | 处理 |
|------|------|
| ✅ 全过 | 进 C3 |
| ❌ 失败 | **不要改测试让它过**。先判测试错 vs 实现 bug；测试错改测试；实现 bug **回 Phase B 修复** 再重跑 |
| ⚠️ 工具/环境问题 | 输出诊断给用户，不强行降级手测 |

### C2 输出

```
🤖 自动化测试结果
- A 类: tests/e2e/T-x.x.x-<slug>.spec.ts (N 用例) → N 通过
- B 类: [逐项关键 PASS 证据]
- 覆盖 C1: VC-1 ~ VC-N 全绿
```

## Phase C3 — 手测清单（仅 🧑 项落地，可短路）

### 短路条件

扫 C1 清单 🧑 项数（含 🤖+🧑 的 🧑 部分）：

- **🧑 项数 = 0** → **完全跳过 C3 不创建文件**，C4 输出"全部已自动化通过；0 项需人手测"
- **≥ 1** → 创建手测清单

### 创建手测清单

路径：`tests/manual/T-x.x.x-<slug>.md`

格式：

```markdown
# T-x.x.x 手测清单

> 本 task 共 M 项需人测。每项独立成节，须命中"手测硬性条件清单"至少一条。

## M-1: {{验证项}}

- **命中条件**: #1 视觉 / #3 真服务 / ...
- **操作步骤**:
  1. {{步骤 1}}
  2. {{步骤 2}}
- **预期**: {{...}}
- **结果**: [ ] 通过 / [ ] 失败

### Bug 反馈区（如失败）
{{用户填写}}

## M-2: ...

（同上）

## 修复历史（如有）

{{修复后追加：日期 + 修复内容}}
```

如 C2 因缺工具被跳过：原 🤖 写入手测清单顶部加 `> ⚠️ 本应自动化，因缺 X 工具降级`，末尾建议初始化命令。

## Phase C4 — 暂停等待

### 标记暂停

Edit `PROGRESS.md`：当前任务状态改 `[⏸️]`。

### 输出按 🧑 项数选模板

**模板 A（🧑=0，C3 短路）**：

```
⏸️ T-x.x.x 已完成实现 — 全部 🤖 通过，0 项需人手测

🤖 C2 结果:
- A 类: [路径 N/N]
- B 类: [关键 PASS 证据]

无 🧑 手测项（无视觉 / 真机 / 真服务 / OS 级 / 主观判断）。
回复"测试通过"我执行 Phase D。
```

**模板 B（🧑≥1）**：

```
⏸️ T-x.x.x 已完成实现，进入验证

🤖 C2: N/N 通过
🧑 C3: M 项待人测 — tests/manual/T-x.x.x-<slug>.md（每项已注明命中硬性条件 #N）

每条勾 [通过] 或 [失败]。
- 全过 → 回复"测试通过"，我执行 Phase D
- 失败 → 清单 Bug 反馈区填细节，回非"测试通过"内容（如"修复 M-2"），
  我读清单 → 修复 → 重跑 C2 → 刷新 C3 → 再次暂停
```

### Bug 反馈循环

用户回非"测试通过"时：
1. Read 清单提取所有 [x] 失败 + Bug 反馈
2. 结合用户对话定位根因 → 回 Phase B 修复（必要时更新 spec / memory draft）
3. 重跑 C2
4. 刷新 C3：重置勾选，**保留 Bug 反馈作修复记录**追加到末尾「修复历史」节
5. 再次进 C4

**不进入下一任务**，直到用户明确回复"测试通过"。

---

# Phase D — 三向同步（PROGRESS + spec + history + Memory）

## D.1 同步 PROGRESS.md

Edit `08-stories/PROGRESS.md`：
- 任务行 `[ ]` → `[✅]` + 日期
- **不**在任务行后嵌套追加冗长摘要
- 检测：若本任务是该 sprint 最后一个 `[✅]` → 触发 sprint 归档（D.5）

## D.2 同步 story 文件

Edit `08-stories/T-x.x.x.md`：
- 头部 `> 状态: DRAFT/IN_PROGRESS` → `COMPLETED`
- 任务清单（如有）`- [ ]` → `- [x]`
- 头部 metadata 区 ≤ 30 行硬限

## D.3 追加 history.md

文件：`08-stories/T-x.x.x.history.md`（同 sprint 共享：`08-stories/S-1.history.md` 也可，按团队偏好）

格式（≤ 200 字段公式不可省）：

```markdown
<a id="t-XXX"></a>
## T-x.x.x ✅ {{date}} — {{一句话亮点 ≤ 30 字}}

- 文件: 新增 N / 修改 M（关键 ≤ 3 个，逗号分隔）
- 测试: 自动化 N / 手测 M（核心通过证据 ≤ 30 字）
- 门禁: format ✓ / lint N / type N / test N / build N
- Memory: 新增 N 条 / 0 条 + 类别（≤ 30 字）
- 备注: [可选 ≤ 30 字 — 仅当反直觉]
```

锚点 ID 规则：`T-1.1.7` → `t-117`（去点号）。

## D.4 同步 Memory（三步）

1. **Draft 定稿**：B.3 写的 draft memory → status 改 `active`，补全模板字段
2. **主索引追加**：`MEMORY.md` 滚动窗口顶部追加：
   ```
   - [<file-stem>](<cat>/<file>.md) — T-x.x.x
   ```
   （**≤ 80 字符硬限**）
   若 `MEMORY.md` 已 20 条 → 把最早一条移到 `memory/.index.md` 对应类别
3. **推翻处理**（如新决策推翻旧的）：
   - 旧文件状态字段标 `superseded-by-<新文件名>`
   - 索引行末追加 `(已废弃)`
   - **不删除**旧文件

## D.5 Sprint 归档（仅当本任务是 sprint 最后一个）

1. 创建 `08-stories/archive/S-{{n}}.md`：复制 PROGRESS.md 当前 sprint 全部内容
2. 清空 `PROGRESS.md` 当前 sprint 节，准备 S-{{n+1}}
3. 在 `00-state.md` 决策快照表追加：`Sprint S-{{n}} ✅ {{date}} | 完成 N 任务`

## D.6 更新 09-implementation.md 索引

Edit `<cwd>/product/<slug>/09-implementation.md`（如不存在则 Write）：

```markdown
# 09 Implementation Progress Index

> 实时进度索引 — 覆盖式更新

## 当前 Sprint S-{{n}}: {{N}}/{{M}}

最近完成:
- T-x.x.x ✅ {{date}} — {{亮点}}
- T-x.x.x ✅ {{date}} — {{亮点}}
- ... (最近 5 条)

详见 08-stories/PROGRESS.md
```

## D.7 提示用户 `/clear`

```
✅ T-x.x.x 同步完成
   PROGRESS [✅] / story COMPLETED / history 追加 / Memory N 条定稿

进度: X/Y 任务完成

🌬️ 建议：运行 /clear 清空对话上下文，然后 /dream-weaving implement 进入下一个任务。
原因：每个任务的上下文是独立的，clear 后从文件重读能避免上下文腐烂。
```

---

## 跨任务状态恢复（用户 /clear 后回来）

用户 `/clear` 后再 `/dream-weaving implement`：

1. 主 SKILL.md 路由识别 `implement` 参数
2. 加载本 stage 文件
3. 执行 Phase A.1 找下一个 `[ ]` 任务
4. **完全不依赖任何会话历史**，只靠文件系统重建上下文

这是 dream-weaving 跨会话不丢失的核心机制。

---

## 反例

- ❌ 跳过 Phase A 直接改代码
- ❌ Phase B 写一半就标 `[✅]`（必须 D 完成才能勾）
- ❌ Phase C2 失败了改测试让它过
- ❌ C3 把 🤖 项也塞进手测清单
- ❌ Phase C4 没等"测试通过"就进 Phase D
- ❌ 写完 memory draft 就标 active（必须 D 才定稿）
- ❌ history 单段 > 200 字（必须强制截断）
- ❌ MEMORY.md 单条 > 80 字符
- ❌ 多个任务一次提交（一个 task 一次 commit）
