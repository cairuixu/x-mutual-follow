# 推特互关榜 · X Mutual Follow

官方账号：@cairuixu
计划域名（Cloudflare Pages 默认子域）：https://x-mutual-follow.pages.dev

## 现在这一版是什么

Cloudflare Pages 纯静态原型（零构建、免费层可跑），已实现用户要的全部流程：

- 登录并登记：handle、昵称、简介/关注需求、领域标签，登记后进入榜单池
- 互关榜：每次刷新页面、每次点“换一换”都重新随机洗牌，每批 12 个，不含自己
- 双向状态：已互关 / 我已关注对方 / 对方已关注我 / 未互关，四种徽章
- 关注流程：点“去关注”打开对方推特主页手动关注，回来再点一次确认记录
- 我的主页：列出我关注过的账号，右边显示对方是否也关注了我
- 门槛：必须先关注 @cairuixu + 发布宣传推文，两项通过才解锁关注按钮

演示版说明（验收前请先读）：静态页的数据只存在访客本机 localStorage，
A 登记的数据 B 的浏览器看不到。这是 Pages 纯静态的固有限制，不是 bug。

## 固定宣传推文

发现一个推特互关神器 🤝 登记账号就能上互关榜，随机刷出新朋友，一键看谁回关了你！先关注 @cairuixu，发布这条推文解锁互关榜 👉 https://x-mutual-follow.pages.dev #推特互关 #互关

正式版判定：用户近期推文里含有本站链接即算通过。

## Cloudflare Pages 免费部署（GitHub 导入路线，用户已用过同款流程）

1. 把本目录全部文件推到 GitHub 公开仓库，例如 `cairuixu/x-mutual-follow` 的 `main` 分支
2. Cloudflare Dashboard → Pages → Create application → Pages → Connect to Git
3. 选该仓库，Production branch 选 `main`
4. Framework preset 选 None，Build command 留空，Build output directory 填 `/`
5. Save and Deploy，得到 `https://x-mutual-follow.pages.dev`（项目名不同则子域不同，
   此时要同步改 index.html 里的 SITE_URL 和宣传推文里的链接）

免费层注意：Pages 免费版足够跑这个静态站；自定义域名、后续 Workers/D1 也都在免费额度内起步。

## 升级为真正的多用户站（下一步，不是本版范围）

要让所有访客互相看到登记数据、并用推特 API 真实核查关注关系，需要：

1. 推特开发者应用：申请 Twitter API v2，OAuth 2.0 回调填本站域名，
   权限 scopes：`tweet.read users.read follows.read offline.access`
2. 后端改用 Cloudflare Workers + D1（免费层）：
   - 表 users（handle、昵称、简介、标签、登记时间）
   - 表 follows（本站记录的关注关系）+ 门槛表（official_followed、promo_tweet_id）
   - 榜单接口：服务端 `ORDER BY RANDOM() LIMIT 12`，每次请求重新随机
   - 核查接口：用用户 access token 调 `GET /2/users/me/following` 与
     `GET /2/users/me/followers`（分页），交集即互关；门槛核查为 following 列表含
     cairuixu 的用户 ID，且近期推文含本站链接
3. 前端把 localStorage 读写换成上述接口；关注仍建议手动在推特完成，
   不要用 `follows.write` 自动批量关注，容易触发风控。

## 合规提醒

- 本站不自动关注、不刷量；请遵守推特关注上限与反垃圾规则。
- 被关注方账号为公开账号时才能核查；受保护账号的关注关系 API 可能不可见。
