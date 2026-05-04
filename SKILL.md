---
name: dream-weaving
description: 把模糊想法编织成产品的 14 阶段全流程引导。当用户说"我想做一个 app/工具/小程序"、"有个想法但不知道怎么开始"、"帮我把这个 idea 实现出来"、"从零开始做一个产品"、"想做但不会写代码"、"vibe coding"、"idea to product"、"织梦"、"想到就做"、"product genesis"、"我想做点什么但还没想清楚"时必须使用此 skill。它强制走完 Idea Crystallization → Market Research → Competitive Analysis → JTBD → PRD → Design System → Tech Architecture → Story Breakdown → Implementation（自带 Phase A-D 微循环）→ Validation → Release → Launch → Growth 的完整路径，而非直接出代码。每个阶段产出独立 markdown 文档持久化到 `<cwd>/product/<slug>/`，跨会话不丢失。**自适应用户技术水平**：完全零基础（不会写代码）、PM/设计师（懂产品不懂代码）、工程师都可用。**完全自包含**，不依赖任何其他 skill，可独立移植到任意 Claude Code 环境。**主动在关键决策点询问是否进入 Claude Code Plan Mode**做结构化规划。用法：/dream-weaving <想法> | /dream-weaving next | /dream-weaving status | /dream-weaving implement | /dream-weaving back <phase> | /dream-weaving switch <slug> | /dream-weaving launch | /dream-weaving growth
disable-model-invocation: true
---

# Dream Weaving — 把想法织成产品

根据 `$ARGUMENTS` 执行 14 阶段产品创造全流程。

## 核心理念（必读，决定一切行为）

1. **Discovery First**：阶段 1-7 不写一行代码，先把 Idea / 调研 / 竞品 / JTBD / PRD / 设计 / 架构 全部想清楚，再有 Discovery Gate 才能进编码
2. **Artifact-First Memory**：所有产物落到 `product/<slug>/` 文件，不靠 chat 上下文记忆
3. **Progressive Disclosure**：本主文件只承载路由 + 全局规则；具体阶段提问/模板/反例在 `stages/0N-*.md` 按需 Read
4. **苏格拉底优先于生成**：每阶段进入时先 1-3 轮 AskUserQuestion 澄清，再产出文档；模糊回答必追问
5. **自适应用户水平**：所有解释、推荐、提问粒度，必须先读 `product/<slug>/00-state.md` 的 `## profile` 节再决定
6. **Plan Mode 主动触发**：在关键决策点（见下文「Plan Mode 触发条件」）AskUserQuestion 询问是否进入 Claude Code Plan Mode

---

## 阶段路由（`$ARGUMENTS` → 行为）

| 参数 | 行为 | 加载的 stage 文件 |
|------|------|------------------|
| 无 | 列出 `product/` 下所有产品 + 当前阶段 + 引导用户 | 无 |
| `<想法描述>` | 创建 slug、初始化目录、Phase 0 + Phase 1 | `stages/00-onboarding.md` → `stages/01-idea.md` |
| `next` | 推进到下一阶段（前置 gate 通过才能推进） | 对应 `stages/0N-*.md` |
| `back <phase>` | 回退某阶段修改 | 对应阶段文件 |
| `status` | 显示当前产品的 14 阶段进度表 | 无（只读 `00-state.md`） |
| `implement` | 进入 Phase 9，执行下一个 `[ ]` story | `stages/09-implementation.md` |
| `launch` | 触发 Phase 12-13 上线流程 | `stages/12-release.md` + `stages/13-launch.md` |
| `growth` | 触发 Phase 14 营销增长 | `stages/14-growth.md` |
| `switch <slug>` | 切换当前活跃产品（多产品并行时） | 无 |

### 路由动作总则

进入任何阶段前，**先**：
1. 读 `product/<active-slug>/00-state.md`（必读，知道当前阶段、用户 profile、gate 状态）
2. 读对应 `stages/0N-*.md`（progressive disclosure，按需加载）
3. 该阶段如声明"相关 memory"，读 `product/<slug>/MEMORY.md` + 命中文件
4. 严格按 stage 文件中的步骤执行

**严禁**：跳过 stage 文件直接根据本主文件的概述就开始执行——主文件只是路由，具体规则在 stage 文件里。

---

## 14 阶段速查表

| Phase | 名称 | Stage 文件 | 主要产物 |
|-------|------|-----------|---------|
| 0 | Onboarding | `stages/00-onboarding.md` | `00-state.md` 的 profile 节 |
| 1 | Idea Crystallization | `stages/01-idea.md` | `01-idea.md` |
| 2 | Market Research | `stages/02-research.md` | `02-research.md` |
| 3 | Competitive Analysis | `stages/03-competitors.md` | `03-competitors.md` |
| 4 | Problem / JTBD | `stages/04-jtbd.md` | `04-jtbd.md` |
| 5 | PRD / MVP Scope | `stages/05-prd.md` | `05-prd.md` |
| 6 | Design System | `stages/06-design.md` | `06-design.md`（可选 `06-design-preview.html`） |
| 7 | Tech Architecture | `stages/07-architecture.md` | `07-architecture.md` |
| ⛩️ | **Discovery Gate** | （在 Phase 7 末尾触发） | `00-state.md` 写入 `gate-passed: <date>` |
| 8 | Story Breakdown | `stages/08-stories.md` | `08-stories/PROGRESS.md` + `08-stories/T-x.x.x.md` |
| 9 | Implementation | `stages/09-implementation.md` | 代码 + commits + memory |
| 10 | QA / Testing | （集成在 Phase 9 的 Phase C） | tests/ + 手测清单 |
| 11 | User Validation | `stages/11-validation.md` | `11-validation.md` |
| 12 | Release / Versioning | `stages/12-release.md` | `12-release.md` + CHANGELOG |
| 13 | Launch / Distribution | `stages/13-launch.md` | `13-launch.md` |
| 14 | Growth / Marketing | `stages/14-growth.md` | `14-growth.md` |

> 注：Phase 10 不独立成 stage 文件，QA 流程整合在 Phase 9 的 Phase C 里。

---

## 文件持久化结构（产物位置）

每个产品一个目录，全部在 `<cwd>/product/<slug>/`：

```
<cwd>/product/<slug>/
├── 00-state.md                  # 当前阶段 + profile + 决策快照（≤ 200 行）
├── 01-idea.md ~ 07-architecture.md   # 思考阶段产物（每份 ≤ 60KB）
├── 08-stories/
│   ├── PROGRESS.md              # 任务状态板（≤ 200 行 / ≤ 8KB）
│   ├── T-x.x.x.md               # 单个 story
│   └── T-x.x.x.history.md       # 任务完成快照（每段 ≤ 200 字）
├── 09-implementation.md         # 实现总进度索引（覆盖式更新）
├── 11-validation.md ~ 14-growth.md
├── MEMORY.md                    # 主索引 FIFO 滚动 20 条（≤ 60 行）
└── memory/
    ├── decisions/<topic>.md
    ├── pitfalls/<topic>.md
    ├── conventions/<topic>.md
    ├── integrations/<topic>.md
    └── .index.md                # 全量索引（懒读）
```

**slug 命名规则**：英文 kebab-case，从用户描述自动派生（如"帮我妈记血压" → `bp-tracker-mom`）。冲突时追加 `-2`、`-3`。

**模板位置**：`~/.claude/skills/dream-weaving/templates/state.md` / `memory-index.md` / `progress.md` / `story.md`。初始化新产品目录时直接 Read 模板内容、替换 `{{...}}` 占位符、Write 到产品目录。

---

## 全局大小硬限（防 context 腐烂）

| 文件 | 上限 | 超限处理 |
|------|------|---------|
| `00-state.md` | ≤ 200 行 | 决策快照表只保最近 20 行，更早的删 |
| 每阶段思考文档 | ≤ 60KB | 拆分为 `0N-<topic>.md` 多文件 |
| `PROGRESS.md` | ≤ 200 行 / ≤ 8KB | 完成 sprint 整体归档到 `08-stories/archive/<sprint>.md` |
| `MEMORY.md` | ≤ 60 行（≤ 20 条） | 最早一条移到 `memory/.index.md` 对应类别 |
| 任意 metadata 头 | ≤ 30 行 | — |
| history 单段 | ≤ 200 字 | — |
| MEMORY.md 单条 | ≤ 80 字符 | — |

---

## 全局规则（每阶段都适用）

### 规则 1：苏格拉底式追问

每个阶段进入后，**必须先 AskUserQuestion 1-3 轮再产出 markdown**。规则：
- 不问显而易见的问题
- 主动提出反方观点 / 用户没想到的边界
- 用户回答模糊则追问具体场景（不要满足于"差不多"、"应该是"）
- 维度覆盖完才动笔写产物
- 每次最多问 1-2 个问题（用 AskUserQuestion 的 multi-question 形式）

### 规则 2：自适应用户水平

每个阶段开始时**强制 Read `product/<slug>/00-state.md` 的 `## profile` 节**。根据 profile 调整：
- **编程"无"** → 任何代码概念出现前必须先用类比解释（如「数据库 = Excel 表」、「API = 服务员」）
- **编程"无"** → 默认推荐零代码或极低代码方案；不主动给 React/SQL/Docker 这类术语
- **编程"会一点"** → 给术语时附 1 句解释；推荐"半托管"方案（如 Vercel + Supabase）
- **编程"熟练"** → 直接用术语对话，跳过基础解释
- **设计"无"** → Phase 6 推荐成熟现成 design system（Tailwind + shadcn / Material）
- **设计"专业"** → Phase 6 给完整 DESIGN.md 模板，不强加风格
- **产品"无"** → 每阶段产物先用 1 段大白话总结再进结构化文档
- **产品"PM"/"独立开发者"** → 直接用 PRD/JTBD/user story 等术语

### 规则 3：Plan Mode 主动触发

**主动 AskUserQuestion 询问用户**是否切到 Claude Code Plan Mode 的场景：

| 触发场景 | 询问时机 |
|---------|---------|
| Phase 7 → 8 切换前 | Phase 7 产物写完，准备进 Phase 8 任务拆分时 |
| Phase 8 内拆分某 story 时，story 涉及 ≥ 5 个文件 / 多模块 / 数据迁移 / 不可逆操作 | 该 story 写到 `08-stories/T-x.x.x.md` 之前 |
| Phase 9 即将执行某 story 且关键文件 ≥ 5 个 | Phase A 读完上下文，进 Phase B 之前 |
| Phase 13 涉及不可逆上架（App Store 审核 / npm 发布）| 触发上架命令前 |

询问模板：

> 这个步骤涉及 [具体原因，如"6 个文件改动 + 数据库迁移"]，建议进入 Claude Code Plan Mode 做一次结构化规划再执行。要切吗？
>
> 如果用户答"是"：提示用户按 `Shift+Tab` 进入 Plan Mode，等待用户确认进入后再继续。
> 如果用户答"否"：在 `MEMORY.md` 写 1 条 `decision` 类型 memory 记录这个跳过决定。

**永远不要**强制切换；**永远只**询问。

### 规则 4：阶段门禁

- **Phase 0 → 1 门禁**：`00-state.md` 的 profile 节 4 项必填全部完成
- **Phase 1-7 之间门禁**：上一阶段产物文件存在且非空
- **⛩️ Discovery Gate（Phase 7 → 8）**：用户必须显式回复"开始实现 / start building / 实现吧"等明确意图，skill 才在 `00-state.md` 写入 `gate-passed: <date>`，此后 `next` 才进 Phase 8。**严禁**自动推进。
- **Phase 8 → 9 门禁**：`PROGRESS.md` 至少有 1 条 `[ ]` 任务
- **Phase 9 → 11 门禁**：所有任务 `[✅]`
- **Phase 11/12/13/14**：用户主动触发对应命令，不自动推进

### 规则 5：Memory 写入触发

Phase 9 实施过程中（详见 `stages/09-implementation.md` 的 Phase B 步骤），如下 4 类信息触发写 memory：

| 类别 | 触发 | 路径 |
|------|------|------|
| decision | ≥ 2 方案非显而易见取舍 | `memory/decisions/<topic>.md` |
| pitfall | 第三方/系统非显而易见行为 | `memory/pitfalls/<topic>.md` |
| convention | 跨任务隐性约定 | `memory/conventions/<topic>.md` |
| integration | 外部系统版本/协议细节 | `memory/integrations/<topic>.md` |

写入流程：先写 `status: draft`，等 Phase D 同步时改 `status: active`。

### 规则 6：禁止反模式

- ❌ 在 Phase 1-7 写任何代码
- ❌ Phase 0 没问完 profile 就开始 Phase 1
- ❌ 跳过 Discovery Gate 直接 `/dream-weaving implement`
- ❌ 把代码细节塞进 `0N-*.md` 思考文档（代码归 `08-stories/T-x.x.x.md` 和实际源码）
- ❌ 在主 SKILL.md 里查找具体阶段提问 —— 那些都在 `stages/0N-*.md`
- ❌ 不读 `00-state.md` 就开始任何阶段
- ❌ 在 Phase 9 之前驱动任何 git/npm/pnpm 写操作
- ❌ 单文件超过大小硬限不归档

### 规则 7：可移植性约束

本 skill 的所有 stage 文件 **不得**包含对其他 skill 的引用或调用（如 `/product-research`、`/spec`、`/html-creator`）。所有能力必须自包含在 `stages/` 和 `templates/` 内。

如果某阶段需要用 WebSearch 等内置工具，直接在 stage 文件里写明用法，不要假设其他 skill 已安装。

---

## 无参数行为（列表模式）

`/dream-weaving` 无参数时：

1. Glob 扫描 `<cwd>/product/*/00-state.md`
2. 对每个产品读取头部 metadata 和 `## profile` 节，输出表格：

```
📋 当前 product/ 下的产品（X 个）

| Slug | 当前阶段 | 进度 | Gate | 上次更新 |
|------|---------|------|------|---------|
| bp-tracker-mom | Phase 5 | 5/14 | ❌ | 2026-05-01 |
| team-okr | Phase 9 | 9/14 | ✅ | 2026-05-03 |

➡️ 用 /dream-weaving switch <slug> 切换；用 /dream-weaving <新想法> 创建新产品。
```

3. 如果 `product/` 不存在或为空：

```
🌱 还没有任何产品。

用 /dream-weaving <你的想法> 开始第一个，例如：
  /dream-weaving 我想做一个帮我妈记血压的小工具
  /dream-weaving 给团队做一个 OKR 对齐看板
```

---

## 启动新产品的标准动作（参数为 `<想法>` 时）

1. 从用户描述派生 slug（kebab-case 英文，3-5 词，必要时 AskUserQuestion 确认）
2. 检查 `<cwd>/product/<slug>/` 是否存在；存在则追加 `-2`
3. 创建目录结构（`mkdir -p product/<slug>/memory/{decisions,pitfalls,conventions,integrations}` + `mkdir -p product/<slug>/08-stories`）
4. 复制 4 份模板到产品目录：
   - `~/.claude/skills/dream-weaving/templates/state.md` → `product/<slug>/00-state.md`
   - `~/.claude/skills/dream-weaving/templates/memory-index.md` → `product/<slug>/MEMORY.md`
   - 替换 `{{product_name}}` 为用户的想法摘要、`{{date}}` 为今天日期
5. 在 `00-state.md` 头部写入「一句话定位」（用户的原始想法）
6. **Read `stages/00-onboarding.md`** 进入 Phase 0
7. Phase 0 完成后自动进入 Phase 1（Read `stages/01-idea.md`）

---

## 状态命令的标准动作（参数为 `status` 时）

1. 确认当前活跃 slug（从 `00-state.md` 或 `switch` 设置的最后一个）
2. Read `product/<slug>/00-state.md` 全文
3. 输出渲染后的状态：当前阶段名、进度、profile 摘要、Gate 状态、最近 3 条决策、下一步建议命令

---

## 切换产品的标准动作（参数为 `switch <slug>` 时）

1. 检查 `<cwd>/product/<slug>/00-state.md` 是否存在
2. 不存在 → 列出可选的 slug
3. 存在 → 把 slug 写入 `<cwd>/product/.active`（单行文件），下次 `next` / `status` / `implement` 默认用此 slug
4. 输出该产品的 `status` 摘要

---

## 推进/回退命令的标准动作

`next`：
1. 读 `00-state.md` 找到第一个未勾选的阶段
2. 检查门禁（见「规则 4」）
3. 通过 → Read 对应 stage 文件并执行
4. 不通过 → 提示缺什么，不推进

`back <phase>`：
1. 验证 phase 编号合法（0-14）
2. AskUserQuestion 确认："你即将回到 Phase {{N}}，已经填好的 `0N-<name>.md` 会被你修改但不删除。继续？"
3. 确认 → 把 `00-state.md` 进度勾选回退到该阶段、Read 对应 stage 文件
4. 如回退跨过 Discovery Gate（从 ≥ 8 回退到 ≤ 7），把 `gate-passed` 标记移除

---

## 实现者备忘（仅供 skill 自身执行参考）

- 所有 Read 操作优先用绝对路径（`<cwd>/product/<slug>/...` 或 `~/.claude/skills/dream-weaving/...`）
- 写文件用 Write，修改用 Edit。**禁用** `cat <<EOF`、`echo >`
- AskUserQuestion 是首选交互工具（不是文本提问）。每次最多 4 个问题，每个问题最多 4 个选项
- 时间统一用 ISO 日期 `YYYY-MM-DD`，不用相对时间词
- 遇到任何模糊→追问，不要瞎猜
- 当用户的回答与已有 profile / 已写产物冲突时，**主动指出冲突**并询问以哪个为准
