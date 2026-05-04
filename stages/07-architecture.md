# Stage 07 — Tech Architecture（技术架构）

> **本阶段职责**：根据 PRD + 设计 + 用户技术水平 + 目标平台，给出 V1 的技术栈、数据模型、部署方案。产物是 `07-architecture.md`。
>
> **核心理念**：选熟悉/主流/AI 友好的技术栈，比"最优技术"重要 10 倍。V1 不为未来 1000x 流量做架构。

---

## 入口检查

1. Read `00-state.md` 的 `## profile` 节
2. Read `01-idea.md`、`05-prd.md`（功能 + 数据模型预想）、`06-design.md`（前端选型 hint）
3. 确认 PRD 中"目标平台"已定（Web / 小程序 / iOS / Android / CLI）

---

## 步骤 1：开场 + 推荐路线（按 profile 强分流）

**编程"无"** → 直接给推荐方案（不要列对比让用户选）：

```
进入 Phase 7：技术架构。

你不需要懂下面的术语，我会做选型并解释每个决定意味着什么。
基于你的画像和 PRD，我推荐：

  目标: {{Web / 小程序 / iOS / ...}}
  推荐栈: {{具体栈，如 "Next.js + Supabase + Vercel"}}

意味着什么：
- 你不需要买/管理服务器
- 上线只需要一个命令
- AI 生成代码就能直接用
- 每月成本 0-9 美元（V1 用户少时基本免费）

替代方案我也会列，但建议先用这个。
```

**编程"会一点"**：

```
进入 Phase 7：技术架构。

我会给 2 个方案对比 + 推荐 1 个。可以听我的，也可以告诉我你的偏好。
```

**编程"熟练" / "资深"**：

```
进入 Phase 7：技术架构。

我会列 3 个候选栈 + 决策矩阵，由你拍板。我会重点指出你 PRD 中**绕不开的硬约束**（如离线、并发、数据规模）。
```

---

## 步骤 2：识别硬约束（强制的 7 个维度）

读 PRD 提取或用 AskUserQuestion 补问：

| 维度 | 问题 |
|------|------|
| 平台 | Web / 小程序 / iOS / Android / 桌面 / CLI |
| 离线 | 必须离线可用 / 必须联网 / 偶尔可离线 |
| 多端同步 | 单端 / 多端实时 / 多端最终一致 |
| 数据规模（V1） | < 1k 条 / 1k-100k / > 100k |
| 并发用户（V1） | 1（自用）/ < 100 / 100-10k / > 10k |
| 实时性 | 秒级 / 分钟级 / 小时级 |
| 隐私敏感度 | 敏感数据本地 / 云端加密 / 普通数据 |

如 PRD 没明确，AskUserQuestion 一次性补问 2-3 个最关键的。

---

## 步骤 3：根据约束 + 平台选栈（默认决策树）

### 默认推荐表

| 平台 | 编程经验 | 推荐栈 | 后端 | 部署 | 数据库 | 月成本 |
|------|---------|--------|------|------|--------|--------|
| Web | 无 | Next.js (App Router) + shadcn/ui + Tailwind | 内置 Server Actions | Vercel | Supabase / Neon Postgres | 0-9 USD |
| Web | 会一点 | 同上，可选 + tRPC | 同上 | 同上 | 同上 | 同上 |
| Web | 熟练 | Next.js / Remix / Nuxt 任选 | Hono / Fastify / Server Actions | Vercel / Cloudflare / Fly.io | Postgres / SQLite (Turso) | 0-20 USD |
| 微信小程序 | 无 / 会一点 | 微信原生（WXML/WXSS） | 云开发（云函数+云数据库） | 微信平台 | 云开发数据库 | 月 19.9 起，开发期免费 |
| 微信小程序 | 熟练 | Taro / uni-app（如要复用 Web 代码） | 自有 BaaS / 微信云 | 同上 | 同上 | 同上 |
| iOS | 无 / 会一点 | SwiftUI（不引入第三方） | iCloud / 本地 SQLite | App Store | iCloud / 本地 | 99 USD/年开发者账号 |
| iOS | 熟练 | SwiftUI + Supabase / Firebase | BaaS | App Store | 同上 | 同上 |
| Android | 类比 iOS | Jetpack Compose | Firebase / 自有 | Play Store | Firebase / SQLite | 25 USD 一次性 |
| 桌面 | 无 | 不建议（V1 改成 Web） | — | — | — | — |
| 桌面 | 熟练 | Tauri (web 技术 + Rust 壳) | 本地 / 远端 | GitHub Releases | SQLite | 0 |
| CLI | 任意 | Bun / Node + commander | 无（或调外部 API） | npm | — | 0 |

### 调整规则

- **离线必需** → 数据库选 SQLite/IndexedDB；前端选纯客户端框架
- **多端同步必需** → 必须有云数据库（Supabase / Firebase / 自有）
- **数据规模 > 100k** → 必须 Postgres，避免 SQLite
- **用户 > 1000 估算** → 把"无运维"的部分（Vercel/Supabase 免费档）变更为"按量付费"
- **隐私敏感（医疗/金融）** → 优先全本地存储，禁用第三方分析

---

## 步骤 4：写入 `07-architecture.md`

```markdown
# 07 Tech Architecture

> 创建日期: {{date}}
> 状态: {{DRAFT | LOCKED}}

## TL;DR

V1 用 **{{推荐栈一句话}}**，全套月成本 < {{X}} USD，AI 编程工具友好，可独立部署上线。

## 硬约束识别（来自 PRD + 步骤 2 补问）

| 维度 | 选择 | 含义 |
|------|------|------|
| 平台 | {{...}} | {{...}} |
| 离线 | {{必须 / 不必 / 偶尔}} | 影响数据库选型 |
| 多端同步 | {{...}} | 影响后端选型 |
| 数据规模 V1 | {{...}} | 影响 DB |
| 并发用户 V1 | {{...}} | 影响部署形态 |
| 实时性 | {{...}} | 影响是否需要 WebSocket / 推送 |
| 隐私敏感度 | {{...}} | 影响数据落地策略 |

## 推荐栈（V1）

### 前端

- **框架**: {{Next.js 16 App Router / SwiftUI / 微信原生 / ...}}
- **UI 库**: {{shadcn/ui + Tailwind / Material 3 / WeUI / ...}}
- **状态管理**: {{React Server Components / Zustand / SwiftData / ...}}
- **路由**: {{框架内置 / ...}}

### 后端

- **形态**: {{Server Actions / 独立 API / 云函数 / 仅前端 + BaaS / ...}}
- **运行时**: {{Node.js 24 / Bun / Swift / 云函数环境 / ...}}
- **API 风格**: {{REST / RPC / GraphQL / Server Actions}}

### 数据存储

- **主数据库**: {{Postgres (Supabase) / SQLite (Turso) / IndexedDB / iCloud / ...}}
- **缓存**: {{无 / Edge Config / Redis / ...}}
- **文件**: {{无 / Vercel Blob / S3 / 本地}}

### 部署

- **托管**: {{Vercel / Cloudflare Pages / 自有 / App Store / ...}}
- **CI/CD**: {{Git push 自动部署 / GitHub Actions / Xcode Cloud / ...}}
- **域名**: {{自有域名 / 平台默认子域名}}

### 第三方服务

- **认证**: {{无（单用户）/ Clerk / Supabase Auth / Apple ID / 微信登录 / ...}}
- **支付** (V1 一般无): {{无 / Stripe / 微信支付 / ...}}
- **分析**: {{无（V1 推荐）/ Vercel Analytics / Plausible / ...}}
- **错误监控**: {{无 / Sentry / ...}}

## 替代方案（备选，V1 不一定用）

### 方案 B: {{...}}

- 优势: {{...}}
- 何时考虑切换到 B: {{...}}

## 数据模型设计

> 基于 PRD §数据模型预想 + Phase 6 §关键组件清单 反推。

### 实体关系（粗略 ERD 文字版）

```
User ──── 1:N ──── Record
                     │
                     1:N
                     │
                   Tag
```

### 主要表 / 集合

#### {{表名 1}}

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | UUID | PK | |
| user_id | UUID | FK → User.id | |
| ... | | | |

#### {{表名 2}}

（同上）

### 索引（V1 必需的）

- `Record(user_id, created_at desc)` — 列表查询
- ...

## 架构图（文字版）

```
[用户浏览器/手机]
        ↓ HTTPS
[Vercel Edge / 微信平台 / App Store]
        ↓
[Next.js Server Actions / 云函数]
        ↓
[Postgres / 云开发数据库]
```

## V1 部署 checklist（Phase 13 会用）

- [ ] 域名（如有）：{{...}}
- [ ] 环境变量：{{DATABASE_URL / API_KEYS / ...}}
- [ ] 数据库迁移脚本：{{...}}
- [ ] HTTPS：{{平台默认 / 自有证书}}
- [ ] 备份：{{平台自动 / 手动 / 不必}}

## 安全 / 隐私（V1 最小集）

- [ ] 密码（如有）：哈希存储（bcrypt / argon2）
- [ ] 敏感数据：{{加密策略}}
- [ ] HTTPS 强制
- [ ] CORS 限制（如分前后端）
- [ ] 速率限制：{{V1 一般用平台默认}}
- [ ] 隐私政策页（即使 V1 也要有最简版）

## 性能预期（V1）

| 指标 | 目标 | 检测方法 |
|------|------|---------|
| 首屏加载 | < 2s（4G）| Lighthouse |
| 主要操作响应 | < 500ms | 浏览器 DevTools / 实测 |
| 数据库查询 | < 100ms | 平台监控 |

## 成本预估（V1，1 个月）

- 托管: {{0 USD（免费档）/ ...}}
- 数据库: {{...}}
- 域名: {{12-15 USD/年 / 0}}
- 第三方服务: {{...}}
- **总计**: {{X}} USD/月

## 升级路径（V1 → V2）

> 当 V1 验证后想加什么的可见技术路径。

- 如要加多用户协作: {{切换 / 升级方案}}
- 如要加 AI 功能: {{方案}}
- 如要加移动端: {{方案}}

## 技术债承认（V1 主动留下的）

> 这些"差"是为了快速上线主动选的，明示出来避免以后被骂。

- {{V1 没做单元测试 → 上线后 1 个月内补}}
- {{V1 没做错误监控 → 等用户 > 50 时加 Sentry}}
- ...
```

---

## 步骤 5：回放 + 校对

```
✅ Phase 7 产物已写入 product/{{slug}}/07-architecture.md

技术栈一句话：{{推荐栈摘要}}
月成本预估：{{X}} USD
{{编程"无" 时额外说：}} 这套栈所有部分都不需要你管服务器，AI 帮你生成代码就能跑起来。

---

⛩️  Discovery Gate ⛩️

到这里 Phase 1-7 全部完成：

  ✅ Phase 1 想法澄清
  ✅ Phase 2 市场调研
  ✅ Phase 3 竞品分析
  ✅ Phase 4 JTBD
  ✅ Phase 5 PRD
  ✅ Phase 6 设计系统
  ✅ Phase 7 技术架构

接下来要进入 Phase 8（任务拆分 + 编码），这是一个**单向门**：
  - 进入后，你的所有讨论会更具体到代码 / 测试 / 部署
  - 想改架构/PRD/设计需要回退（/dream-weaving back N）但产物会保留
  - 推荐你先回去通读一遍 01 ~ 07 的产物，看看有没有想改的

我**不会自动**进入 Phase 8。请做一件事：

1. 通读 product/{{slug}}/01-idea.md 到 07-architecture.md
2. 满意后回复"开始实现 / start building / 实现吧"
3. 如想改某阶段，回复 /dream-weaving back N

🔔 进入 Phase 8 前我会主动询问你**是否切到 Claude Code Plan Mode**。
   原因：任务拆分是关键决策点，Plan Mode 能让规划更结构化。
```

---

## 步骤 6：等待 Discovery Gate

**严禁**自动进 Phase 8。**严格**等待用户回复以下任一意图：

- "开始实现" / "start building" / "实现吧" / "go"
- "build it" / "let's code" / "进 Phase 8"

收到后：
1. 在 `00-state.md` 写入 `gate-passed: <YYYY-MM-DD>`
2. **AskUserQuestion**：「即将开始 Phase 8 任务拆分。这是关键决策点，建议进入 Claude Code Plan Mode 做结构化规划。要切吗？」
3. 用户答"是" → 提示用户按 `Shift+Tab` 进入 Plan Mode
4. 用户答"否" → 在 `MEMORY.md` 写一条 `decision` memory 记录跳过决定
5. 进 Phase 8（Read `stages/08-stories.md`）

---

## 步骤 7：Exit gate（Phase 7 内部）

- [x] `07-architecture.md` 写完
- [x] 7 个硬约束维度都有选择
- [x] 推荐栈含前端/后端/数据库/部署/第三方 5 个维度
- [x] 数据模型至少 1 张表 + 主要字段
- [x] V1 部署 checklist 至少 4 项

满足后才能展示步骤 5 的 Discovery Gate 文案。

---

## 反例

- ❌ 给编程"无"用户列 5 个候选栈让 TA 选
- ❌ 推荐 V1 用 Kubernetes / 微服务 / 自建 K8s
- ❌ 跳过硬约束识别，直接选栈
- ❌ 数据模型节空着（"等开发再设计"）
- ❌ 自动进 Phase 8，跳过 Discovery Gate
- ❌ 不主动询问 Plan Mode
