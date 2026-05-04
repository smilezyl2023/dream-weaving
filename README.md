# dream-weaving

> 把模糊想法织成产品的 Claude Code skill — 14 阶段全流程引导，从 idea 到上架到增长。

写给所有用过 Lovable / Bolt / v0 / Replit Agent 之后，**第一版 demo 出得很爽，但第三天就想推倒重来**的人。

## 它解决什么

非编程用户用 AI 编程工具时反复遇到的三件事：

1. **直接出代码反模式** — 工具不会反问"用户换手机数据怎么办"，垃圾进垃圾出
2. **流程碎片化** — PRD 在 Notion，调研在 Perplexity，代码在 Cursor，上架查苹果文档，每一步重新 onboarding AI
3. **跨会话记忆腐烂** — 聊到第 50 轮 AI 已经忘了你第 5 轮定的"用 Postgres 不用 SQLite"

`dream-weaving` 不是又一个出 demo 的工具，**它强制你先想清楚再写**。14 个阶段每个都有产物文档，跨会话不丢失，可独立移植到任意 Claude Code 环境。

## 14 个阶段

| Phase | 名称 | 这一步在回答什么 |
|-------|------|-----------------|
| 0 | Onboarding | 你是谁？编程/AI/产品/设计四个维度的水平 |
| 1 | Idea Crystallization | 你究竟想解决什么？给谁？为什么是你做？ |
| 2 | Market Research | 这个需求真的存在吗？市场多大？ |
| 3 | Competitive Analysis | 现有方案是什么？你的差异在哪？ |
| 4 | Problem / JTBD | 用户在什么场景下「雇佣」你这个产品？ |
| 5 | PRD / MVP Scope | 第一版只做什么？砍掉什么？ |
| 6 | Design System | 视觉风格 + 关键流程的设计语言 |
| 7 | Tech Architecture | 技术栈、数据模型、部署形态 |
| ⛩️ | **Discovery Gate** | 用户必须显式确认才能进实现阶段 |
| 8 | Story Breakdown | 拆成可独立交付的 story（建议触发 Plan Mode） |
| 9 | Implementation | Phase A→B→C→D 微循环执行编码 |
| 10 | QA / Testing | 集成在 Phase 9 的 Phase C 中 |
| 11 | User Validation | 真实用户反馈、是否解决问题 |
| 12 | Release / Versioning | SemVer 版本号、CHANGELOG、git tag |
| 13 | Launch / Distribution | App Store / npm / 微信小程序 / Product Hunt 上架 checklist |
| 14 | Growth / Marketing | landing page、SEO、内容、留存指标 |

## 安装

```bash
git clone https://github.com/smilezyl2023/dream-weaving.git ~/.claude/skills/dream-weaving
```

需要 [Claude Code](https://claude.com/claude-code) 已安装。

## 使用

```bash
cd ~/Documents/Projects/    # 你想放产品产物的地方
claude
```

```
# 启动新产品
> /dream-weaving 我想做一个帮我妈记录每天血压的工具

# 推进下一阶段
> /dream-weaving next

# 查看进度（跨会话也能用）
> /dream-weaving status

# 进入实现阶段（Discovery Gate 通过后）
> /dream-weaving implement

# 回退某阶段修改
> /dream-weaving back 5

# 多产品并行
> /dream-weaving switch <slug>

# 上架 / 增长
> /dream-weaving launch
> /dream-weaving growth
```

## 产物在哪

每个产品一个目录，全部在你启动 claude 时所在的工作目录下：

```
<cwd>/product/<slug>/
├── 00-state.md                  # 当前阶段 + 用户 profile + 决策快照
├── 01-idea.md ~ 07-architecture.md   # 思考阶段产物
├── 08-stories/
│   ├── PROGRESS.md              # 任务状态板
│   └── T-x.x.x.md               # 单个 story 详情
├── 11-validation.md ~ 14-growth.md
├── MEMORY.md                    # 主索引（FIFO 滚动 20 条）
└── memory/
    ├── decisions/               # 重要决策
    ├── pitfalls/                # 踩过的坑
    ├── conventions/             # 跨任务隐性约定
    └── integrations/            # 外部系统集成细节
```

**关键大小硬限**（防 context 腐烂）：`00-state.md` ≤ 200 行 / 每阶段产物 ≤ 60KB / `MEMORY.md` 滚动 20 条 / history 单段 ≤ 200 字。

`/clear` 之后 `/dream-weaving status` 能从 `00-state.md` 完整重建上下文，不依赖任何会话历史。

## 4 个关键机制

**Progressive Disclosure** — 主 SKILL.md 只承载路由 + 全局规则（285 行），每个阶段拆成独立文件按需 Read。单次会话只加载主入口 + 当前阶段，避免全量加载烧 token。

**Discovery Gate** — 阶段 1-7 完成后 skill **不会自动**进 Phase 8，必须用户显式回复"开始实现"。这是为了对抗"写完就以为读过"的心理错觉。

**Phase A→B→C→D 微循环** — 阶段 9 不是"按 PRD 写代码"，每个 story 都按 4 阶段执行：A 读上下文 → B 实现 → C 分层验证（C4 必须暂停等用户）→ D 三向同步（PROGRESS + story + memory）。Claude 永远不会自己说"测试通过下一个"。

**自适应用户水平** — Phase 0 收集 4 维 profile（编程/AI 工具/产品/设计经验），所有阶段进入时强制读 profile 调整解释深度、术语门槛、推荐路线。零基础推 no-code 路线，工程师直接给 self-host 方案。

## 适合 / 不适合

**适合**：

- 有想法、想认真做、愿意花一周以上的人
- 不接受"AI 直接出 demo 我看着改"这种生产方式的人
- 之前用 Lovable / Bolt 做过 demo 但没坚持下去的人

**不适合**：

- 想 30 分钟内出 demo 给老板看的（去用 v0）
- 已经有完整 PRD + 设计稿、只缺执行的（直接用 Cursor / Claude Code）
- 不想读自己写的文档的（这不是工具能解决的问题）

## 边界

skill 不替你写代码（Phase 9 引导 AI 编程工具写）；不替你做调研（Phase 2 给搜索词，搜索是 WebSearch + 你自己读）；不替你做设计；不替你做营销。**它是脚手架，不是替身**。14 个阶段每个都需要你的输入和判断。

## 项目结构

```
.
├── SKILL.md                       # 主入口（285 行）
├── stages/                        # 14 个阶段详细流程
│   ├── 00-onboarding.md
│   ├── 01-idea.md
│   ├── ...
│   └── 14-growth.md
├── templates/                     # 4 个产物模板
│   ├── state.md
│   ├── memory-index.md
│   ├── progress.md
│   └── story.md
├── CLAUDE.md                      # 开发约定（修改 skill 的同步流程）
└── README.md
```

总规模约 3900 行，零外部 skill 依赖，单文件夹拷走即可移植。

## 贡献

issue / PR 都欢迎。修改 skill 的具体工作流见 `CLAUDE.md`（项目目录是 source of truth，需同步到 `~/.claude/skills/dream-weaving/`）。

## License

MIT
