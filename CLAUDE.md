# dream-weaving Skill 开发约定

## 项目定位

这是 `dream-weaving` Claude Code skill 的**源码仓库**。

- **Source of Truth**：本项目目录 (`/Users/smilezyl/Documents/Projects/dream-weaving/`)
- **运行时位置**：`~/.claude/skills/dream-weaving/`（Claude Code 实际加载这里）

任何对 skill 的修改，**必须先改本项目目录的文件，再同步到运行时位置**，不要反向。

## 修改工作流

```
本项目目录改文件 → 验证内容 → 同步到 ~/.claude/skills/dream-weaving/ → 用 /dream-weaving 验证 → git commit + push
```

### 同步命令（项目目录 → 系统 skill 目录）

```bash
rsync -av --delete \
  --exclude='.git' \
  --exclude='.gitignore' \
  --exclude='CLAUDE.md' \
  --exclude='README.md' \
  --exclude='article.md' \
  --exclude='.DS_Store' \
  ./ ~/.claude/skills/dream-weaving/
```

`--delete` 会删除目标目录里源目录已不存在的文件——保证两边一致。运行前确认 `pwd` 在本项目根目录。

## 目录结构

```
.
├── SKILL.md                       # 主入口（路由 + 全局规则 + 加载指引）
├── stages/                        # 14 阶段详细流程
│   ├── 00-onboarding.md
│   ├── 01-idea.md
│   ├── ...
│   └── 14-growth.md
├── templates/                     # 4 个产物模板
│   ├── state.md
│   ├── memory-index.md
│   ├── progress.md
│   └── story.md
├── CLAUDE.md                      # 本文件（不同步到 skill 目录）
├── .gitignore                     # 排除 article.md 等
└── article.md                     # 本地草稿，不入版本控制
```

注意：

- Phase 10（QA）整合在 Phase 9 的 Phase C 中，所以 `stages/` 没有 `10-*.md`
- `templates/` 模板里的 `{{...}}` 是占位符，实际产物在 `<cwd>/product/<slug>/` 由 skill 写入

## 修改 skill 时的硬约束

来自 SKILL.md 的全局规则，改任何 stage 文件都要遵守：

1. **Progressive Disclosure**：主 SKILL.md 只承载路由，具体规则放 stages/，不要把 stage 内容塞回主文件
2. **自包含**：禁止引用其他 skill（如 `spec`）。grep 检查：
   ```bash
   grep -rn "spec skill\|@spec\|/.claude/skills/spec" stages/ templates/ SKILL.md
   ```
   只能匹配主 SKILL.md 里"应不引用其他 skill"这条规则文本本身，其他匹配都是 bug
3. **大小硬限**：
   - `00-state.md` ≤ 200 行
   - `MEMORY.md` 滚动 20 条
   - history 单段 ≤ 200 字
   - 每阶段产物文档 ≤ 60KB
4. **Discovery Gate** 不可绕过：Phase 7→8 必须 `gate-passed` 标记
5. **C4 不可省略**：Phase 9 验证阶段必须暂停等用户确认，绝不自动推进

## Commit 规范

- 用英文 + Conventional Commits（feat/fix/docs/refactor/chore/test）
- 例：
  - `feat(stages): add Phase 6 design system preview generation`
  - `fix(stages/09): tighten Phase C4 wait condition`
  - `docs(SKILL): clarify routing parameter semantics`
  - `refactor(templates): split state.md profile section`

## 常见任务速查

**新增一个阶段子规则** → 改 `stages/0N-*.md` → 同步 → `/dream-weaving back N` 验证

**新增全局规则** → 改 `SKILL.md` 的"全局规则"节 → 同步 → 任意阶段验证

**调整模板结构** → 改 `templates/*.md` → 同步 → 新建 `/dream-weaving 测试` 验证模板渲染

**重命名/删除阶段文件** → 改 `SKILL.md` 的阶段路由表 → 改文件名 → 同步（必须用 `--delete` 才会删旧文件）→ 验证

## 调试 tips

- skill 没生效？先确认 `~/.claude/skills/dream-weaving/SKILL.md` 是否同步了最新内容
- 提示词改了但行为没变？Claude Code 缓存 skill metadata，重启 session 或 `/clear`
- 路由参数行为不对？检查 SKILL.md 的"阶段路由"表是否覆盖该参数
