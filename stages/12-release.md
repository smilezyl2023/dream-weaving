# Stage 12 — Release / Versioning（发版与版本管理）

> **本阶段职责**：把验证通过的 V1 打成正式版本，写 changelog，打 tag。产物是 `12-release.md` + 项目根 `CHANGELOG.md`。
>
> **核心理念**：v1.0.0 不是结束，是"开始有责任"的起点。语义化版本（SemVer）+ 清晰的 changelog 让你后面的迭代不混乱。

---

## 入口检查

1. Read `00-state.md`，确认 Phase 11 验证通过
2. Read `08-stories/PROGRESS.md` 全部任务 `[✅]`
3. Read `00-state.md` 的 `code-root` 字段（Phase 8 时定）确认代码位置

---

## 步骤 1：版本号决策

按 SemVer：`MAJOR.MINOR.PATCH`

- **0.1.0** — V1 内测版，预期会有 breaking changes
- **0.x.0** — 公测，功能有调整空间
- **1.0.0** — 第一个公开稳定版，保证 API/UX 向后兼容

AskUserQuestion：

```
问题：V1 起始版本号？
- 0.1.0（推荐 — 给自己留改 API 的余地）
- 1.0.0（V1 已稳定，承诺向后兼容）
- 自定义
```

---

## 步骤 2：检查发版前的硬性条件

输出检查清单，**未达标则停止**：

```
📋 发版前 checklist

代码质量:
- [ ] 所有任务测试通过（PROGRESS [✅]）
- [ ] format / lint / type / test / build 五连门禁通过
- [ ] 无 console.log / TODO / FIXME 残留（grep 检查）
- [ ] 环境变量在 .env.example 有完整说明
- [ ] 隐私政策 / 用户协议（即使最简版）

文档:
- [ ] README.md 有：一句话定位 / 安装 / 启动 / 首次使用
- [ ] CHANGELOG.md 准备就绪（本阶段写）

法律 / 合规:
- [ ] 第三方依赖的 License 兼容
- [ ] 用户数据收集 → 隐私政策声明
- [ ] 如涉及付款 / 医疗 → 相关合规

如有未勾选项，请先补齐再回 /dream-weaving launch。
```

如全部通过，进入步骤 3。

---

## 步骤 3：写 CHANGELOG.md

格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)：

```markdown
# Changelog

本项目所有重要变更记录于此。

格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本 2.0.0](https://semver.org/lang/zh-CN/)。

## [0.1.0] - {{date}}

### 新增

- {{P0 功能 1：F-1 简述}}
- {{P0 功能 2：F-2 简述}}
- ...

### 已知问题

- {{Phase 11 验证发现但接受的小问题}}
- {{从 09-implementation.md 的 "技术债承认" 中提取}}

### 不在本版本

- 多用户协作（V2 候选）
- 数据导出（V2 候选）
- {{从 PRD §不在 V1 范围 提取}}
```

如代码用 git 管理：
1. 用 Bash `git status` 确认无未提交改动
2. 询问用户是否打 tag（**不主动 push**）：

```
是否打 git tag v0.1.0？
- 是 → 我执行 `git tag -a v0.1.0 -m "Release v0.1.0"`
- 否 → 你自己处理
- 还要 push 到 remote？ → 你执行 `git push --tags`
```

---

## 步骤 4：写 `12-release.md`

```markdown
# 12 Release

> 创建日期: {{date}}
> 版本: v{{version}}
> Code root: {{path}}

## 发版决策记录

- **基于**: Phase 11 用户验证通过（{{N 个用户 / {{核心指标达成}}）
- **范围**: PRD §V1 P0 全部完成（{{N}} 个 story）
- **未含**: PRD §V1 P1 的 {{M}} 个功能延后到 v0.2.0

## 发版 checklist 完成情况

（步骤 2 的清单结果，逐项记录）

## CHANGELOG 摘要

新增功能 {{N}} 项。完整见 CHANGELOG.md。

## Git tag

- 已打: v{{version}}（{{commit hash}}）
- 已 push 到 remote: ✅ / ❌

## 下一步

- /dream-weaving next 进入 Phase 13 上架
- 或 /dream-weaving back 9 加新 task
```

---

## 步骤 5：Exit gate

- [x] CHANGELOG.md 存在且含 v{{version}} 节
- [x] `12-release.md` 写完
- [x] 发版前 checklist 全部 ✅
- [x] git tag 已打或用户明确选"否"
- [x] 用户确认进入 Phase 13

---

## 反例

- ❌ 跳过发版 checklist 直接打 tag
- ❌ CHANGELOG 写"修了一些 bug"（必须具体到功能）
- ❌ 主动 git push（必须用户授权）
- ❌ V0 直接打 v1.0.0（让自己被向后兼容承诺套牢）
- ❌ 把没有真实用户验证的版本当 v1.0.0
