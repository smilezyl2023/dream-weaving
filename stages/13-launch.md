# Stage 13 — Launch / Distribution（上架与分发）

> **本阶段职责**：把发版的 V1 上架到对应平台/分发渠道，让真实陌生用户能找到并用上。产物是 `13-launch.md`。
>
> **核心理念**：上架是一次性 checklist + 平台特化要求的执行。不同平台流程差异大，本阶段按 PRD 的目标平台分流出对应清单。

---

## 入口检查

1. Read `00-state.md`，确认 Phase 12 完成（CHANGELOG + git tag）
2. Read `07-architecture.md` §部署 + §第三方服务（拿到平台和域名信息）
3. Read PRD §目标平台

---

## 步骤 1：识别上架平台并加载对应清单

根据 PRD §目标平台，加载下方对应小节。

### 平台清单总览（必选 1+）

- A. **Web（自有域名 + 托管平台）**
- B. **微信小程序**
- C. **iOS App Store**
- D. **Android（Play / 国内应用商店）**
- E. **npm 包**（CLI 工具 / SDK）
- F. **Product Hunt / Hacker News / 其他社区**
- G. **桌面应用**（GitHub Releases / Mac App Store）

---

## A. Web 上架

```
✅ Web 上架 checklist

域名:
- [ ] 已注册域名（推荐：Namecheap / Cloudflare / 阿里云）
- [ ] DNS 指向托管平台（Vercel / Cloudflare Pages / Netlify）
- [ ] HTTPS 自动证书（平台默认）
- [ ] WWW 重定向（www.x.com → x.com 或反之）

部署:
- [ ] 生产环境部署成功（Vercel: vercel --prod）
- [ ] 环境变量已在平台配置（区分 production / preview）
- [ ] 数据库连接字符串走环境变量
- [ ] 测试 production URL 全流程跑通

监控（最小集）:
- [ ] 平台自带日志可访问（vercel logs / cloudflare dashboard）
- [ ] 错误页（404 / 500）有自定义页面
- [ ] 主页 favicon / og:image / meta 描述齐全（社交分享）

SEO（V1 最小）:
- [ ] sitemap.xml
- [ ] robots.txt
- [ ] 主页 meta title / description / og 标签
```

---

## B. 微信小程序上架

```
✅ 微信小程序上架 checklist

资质:
- [ ] 微信小程序账号已认证（个人 / 企业）
- [ ] 类目已选择且符合产品定位
- [ ] 服务器域名已在 mp.weixin.qq.com 后台配置
- [ ] 业务域名 / 业务接口域名已配置

代码:
- [ ] 小程序版本号符合 SemVer（如 0.1.0）
- [ ] app.json 中 pages / window / tabBar 完整
- [ ] 开发者工具"上传"测试通过

提审:
- [ ] 提审版本上传到微信后台
- [ ] 填写测试账号（如有登录）
- [ ] 填写功能描述（500 字内）
- [ ] 提交审核

发布前:
- [ ] 审核通过通知
- [ ] 后台点击"发布"
- [ ] 真机扫码访问验证
```

---

## C. iOS App Store

```
✅ App Store 上架 checklist

账号:
- [ ] Apple Developer 账号年费 99 USD 已支付
- [ ] App Store Connect 中 App 已创建（含 Bundle ID）

资源:
- [ ] App 图标（1024x1024 + 各尺寸）
- [ ] 截图（按设备规格 6.7" / 6.5" / 5.5" 各 ≥ 3 张）
- [ ] App 预览视频（可选）
- [ ] App 名称（中英）
- [ ] 副标题（30 字符）
- [ ] 关键词（100 字符总）
- [ ] 描述（4000 字符）
- [ ] 隐私政策 URL
- [ ] 支持 URL

代码:
- [ ] Bundle ID / 版本号 / Build Number 正确
- [ ] 已开启 App Sandbox
- [ ] 隐私清单（PrivacyInfo.xcprivacy）
- [ ] 第三方 SDK 隐私清单

提审:
- [ ] Xcode Archive → Distribute App → App Store Connect
- [ ] App Store Connect 选择 Build 提交审核
- [ ] 填写审核备注（含测试账号）

发布:
- [ ] 通过审核（一般 1-3 天）
- [ ] 选择"自动发布"或手动
```

---

## D. Android 上架

简版（Google Play）:

```
✅ Google Play checklist

- [ ] Google Play Developer 账号 25 USD 一次性
- [ ] APK / AAB 签名打包
- [ ] Play Console 创建 App
- [ ] 商店列表：图标 / 截图 / 描述
- [ ] 隐私政策 URL（必需）
- [ ] 内容分级问卷
- [ ] 提交审核
```

国内（小米 / 华为 / 应用宝 / vivo / OPPO 各自后台）按需选择。

---

## E. npm 包

```
✅ npm 发布 checklist

代码:
- [ ] package.json 字段齐全：name / version / description / main / types / exports / files / license / repository
- [ ] README.md 完整（安装 / 使用示例 / API）
- [ ] LICENSE 文件
- [ ] .npmignore 排除测试 / 源文件（如打包后）

发布:
- [ ] npm login
- [ ] npm publish --access public（如 scope 包）
- [ ] 验证：在临时目录 npm install 跑通
```

---

## F. Product Hunt / Hacker News 等社区

简版：

```
✅ Product Hunt 发布

- [ ] 准备 thumbnail（240x240）+ gallery 4-5 张
- [ ] 一句话 tagline（60 字符）
- [ ] 描述（260 字符）
- [ ] First comment 草稿（背景故事 / 灵感 / 求反馈）
- [ ] 选好发布日（Tue-Thu 通常流量好；中文产品考虑国内时区）
- [ ] 发布前 24h 通知种子用户准备 upvote 和评论
```

Hacker News：

```
- [ ] Show HN: 标题（"Show HN: <一句话定位>"）
- [ ] 简短自我介绍 + 为什么做这个 + 求反馈
- [ ] 准备应对技术质疑的回复
```

---

## G. 桌面应用

```
✅ 桌面应用分发 checklist

- [ ] 跨平台打包（Tauri build / Electron-builder）
- [ ] macOS：开发者签名 + 公证（Apple notary）
- [ ] Windows：代码签名证书（建议 EV）
- [ ] GitHub Releases：上传安装包 + 写 release note（复用 CHANGELOG）
- [ ] 自动更新（如有需要）：tauri-updater / electron-updater
```

---

## 步骤 2：写入 `13-launch.md`

```markdown
# 13 Launch / Distribution

> 创建日期: {{date}}
> 版本: v{{version}}
> 上架平台: {{A / B / C / D / E / F / G 中的一个或多个}}

## 上架 checklist 完成情况

（按选择的平台逐项记录完成情况，未完成的标 ⏸️）

## 关键链接

- 生产 URL: {{...}}
- App Store: {{...}}
- 小程序: {{微信扫码二维码截图 / 小程序名称}}
- npm: https://www.npmjs.com/package/{{...}}
- Product Hunt: {{...}}
- 仓库: {{...}}

## 时间记录

- 提审时间: {{...}}
- 审核通过: {{...}}
- 实际上架: {{...}}

## 上架后即时验证

- [ ] 用全新设备/账号测试主流程
- [ ] 在不同网络环境（4G / Wifi）测试
- [ ] 真机性能（启动时间 / 主操作响应）
- [ ] 收集前 5 个用户的反馈

## 已知问题

{{上架后短期内不修但需告知的}}

## 下一步

- /dream-weaving next 进入 Phase 14 营销增长
- 或保持现状观察 1-2 周
```

---

## 步骤 3：Exit gate

- [x] `13-launch.md` 写完
- [x] 至少 1 个平台的上架 checklist 全部 ✅
- [x] 关键链接已记录
- [x] 上架后即时验证 全部 ✅

满足后勾选 Phase 13、追加决策快照。

---

## 反例

- ❌ 提审就标完成（要等审核通过 + 实际上架 + 真机验证）
- ❌ 跳过 SEO / 隐私政策 / favicon 这些"小事"
- ❌ npm 发布前不在临时目录验证
- ❌ Product Hunt 发布当天没人配合 upvote
- ❌ 上架后不做即时全流程测试（线上环境与开发常有差异）
