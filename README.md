# X-ui


> **⚠️ 重要提示：一键部署后如果页面显示空白或 "Hello World"，请阅读下方 [一键部署故障排除](#一键部署故障排除)。**

XUI 是一个部署在 **单一 Cloudflare Worker** 的代理节点管理与服务器探针面板。Worker Assets 托管前端和 VPS 安装组件，D1 保存配置、用户、流量和探针数据，Durable Objects 提供实时 WebSocket；无需部署传统面板服务器或额外 Realtime Worker。

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/xiaoli8571/X-UI)

## 一键部署

1. 点击上方 **Deploy to Cloudflare Workers**。
2. 登录 Cloudflare，选择账户并确认 Worker 名称。
3. Cloudflare 会自动创建并绑定 D1 数据库到 `DB`，同时创建实时状态需要的 Durable Objects。不要删除这些 bindings。
4. **首次访问：** 部署成功后直接打开 Worker 地址即可登录。
   - 如果页面正常显示登录界面，说明部署成功。
   - **如果页面显示空白、"Hello World" 或 1101 错误，请参照下方故障排除。**

### 一键部署故障排除

如果部署后页面无法正常加载，请按以下步骤排查：

**① 检查 D1 数据库绑定**
- 进入 Cloudflare Dashboard → Workers & Pages → 你的 Worker (xui)
- 左侧 → **Settings → Variables**
- 在 **D1 Database Bindings** 下找到 `DB`
- 如果没有绑定，点击 **Add binding**：
  - Variable name: `DB`
  - D1 database: 选择或创建一个 D1 数据库
- 绑定后点击 **Deploy** 重新部署

**② 检查 Durable Objects 绑定**
- 同一页面，在 **Durable Object Bindings** 下方应有以下两项：
  - `VPS_PRESENCE` → VpsPresence
  - `DASHBOARD_HUB` → DashboardHub
- 如果缺失，手动添加并重新部署

**③ 检查兼容性标志**
- 在 **Compatibility flags** 中添加：`assets_navigation_prefer_worker`
- 重新部署

**④ 如果仍然显示 Hello World**
- 在 Workers & Pages 页面找到你的 Worker
- 点击进入 → 点击右上角 **Quick Edit**
- 确认编辑器中的代码与实际仓库代码一致
- 如不一致，手动将 `src/worker.js` 内容复制进去，保存并重新部署

预设登录信息：

```text
用户名：admin
密码：admin
```

这是为完整一键部署准备的默认值。首次登录后必须在 Worker 的 **Settings → Variables and Secrets** 将 `ADMIN_PASSWORD` 覆盖为强 Secret 并重新部署。

住宅代理默认**关闭**（`PROXY_USER`/`PROXY_PASS` 为空 = 零轮询零消耗）。

### 住宅代理是什么

> ✅ **功能已开放**（部署后按需启用）：代码、前端 UI、VPS 组件均完整内置，任何账户一键部署后即可使用；默认关闭（零轮询零消耗）。

让节点出口 IP 变成**住宅 IP**（真实家庭宽带 IP）而非机房 IP，用于解锁 Netflix / Disney+ / ChatGPT 等流媒体与风控服务，是代理面板的高溢价卖点。

### 如何启用（部署后按需开启）

1. **VPS 侧**：目标 VPS 执行安装命令装住宅代理组件（`residential-proxy.sh` + proxy-lite），详见下方部署命令区的 `Full Deploy Command`（XUI + 住宅双隧道）。
2. **面板侧**：在 Worker 的 **Settings → Variables and Secrets** 添加独立强 Secret 并重新部署：

```text
PROXY_USER=你的用户名
PROXY_PASS=你的密码
```

3. **前端**：VPS 卡片 → 出口模式选 **住宅 IP 代理**，可切换「全局代理」/「局部代理」；配置后住宅代理自动启用。

### ⚠️ 额度注意事项

- `wrangler.jsonc` 的 vars 里**不要填非空默认值**——vars 里非空值会被当作"已配置"从而启用住宅代理轮询，若没有真实住宅出口账号会持续产生请求，**烧掉免费额度**。
- 正确姿势：仓库保持空值，部署后在 Dashboard 用 **Secret** 类型配置（Secret 不随仓库分发，且单独管理）。
- 不用时在面板「代理池」或 TG 机器人 `/proxy ... off` 一键关闭，VPS 端立即进入低功耗模式，停止轮询。
- 额度基线（内置）：住宅代理关闭 = 零轮询零消耗；启用后 VPS 恢复 **480s 低频轮询**；Agent HTTP 轮询 300s、WebSocket 重连上限 300s、前端兜底轮询 60s、Cron 15 分钟。

## 自定义域名

在 Worker 的 **Settings → Domains & Routes → Add** 中绑定域名或子域名。绑定后直接使用该域名访问面板。

## 本地部署

适用于需要使用已有 D1、固定 Worker 名称或自行维护发布流程的场景。

```bash
git clone https://github.com/xiaoli8571/X-UI.git
cd X-UI
npm install
npx wrangler login
npx wrangler deploy
```

当前 `wrangler.jsonc` 未指定 D1 ID，首次部署会自动创建数据库和实时 Durable Objects。若需要使用已有 D1，在 Cloudflare Dashboard 的 Worker **Settings → Bindings** 中将 `DB` 重绑到目标数据库后重新部署。

生产环境请立即替换默认密码：

```bash
npx wrangler secret put ADMIN_PASSWORD
npx wrangler deploy
```

本地预览：

```bash
npm run dev
```

## 已内置实时服务

实时 WebSocket、Agent 在线状态、即时配置刷新、公开探针实时更新和观众频率自适应均已内置于主 Worker。

部署后无需配置：

- `REALTIME_URL`
- `PAGES_ORIGIN`
- 单独的 Realtime Worker
- 单独的 Realtime D1 或 Durable Objects

## VPS 接入

1. 登录 XUI，进入 **服务器与节点**。
2. 添加 VPS 名称和公网 IP。
3. 复制页面生成的 Full Deploy Command，以 `root` 在 VPS 执行。
4. 等待 Agent 回连后创建节点或使用"8 合 1"批量部署。

支持 XTLS-Reality、Hysteria2、TUIC、Trojan、H2/gRPC-Reality、AnyTLS、Naive、VLESS-Argo、Socks5 与 Dokodemo-door。

## 主要能力

- 多用户、订阅令牌、流量配额和到期管理。
- Mihomo/Clash 订阅导出，包括 AnyTLS。
- CPU、内存、磁盘、网络、TCP/UDP 与线路延迟探针。
- 多种预设探针主题、自定义 CSS 和背景。
- 原生、WARP、住宅代理和手动 SOCKS5 节点出口。
- 可选 Telegram 告警与订阅保护。
- Worker Cron 每 15 分钟检查离线节点。

## 🤖 Telegram 机器人（可选）

内置 Telegram 机器人，支持**失联告警推送**和**远程控制**。

### 配置方法

1. 在 [@BotFather](https://t.me/BotFather) 创建机器人，获得 `bot_token`。
2. 获取你的 Telegram 数字 ID（如向 [@userinfobot](https://t.me/userinfobot) 发送任意消息）。
3. 面板 **设置 → 探针大盘** 填写 `tg_bot_token` 和 `tg_chat_id`，保存后自动注册 Webhook。
4. 设置 Webhook 密钥（防止他人调用）：

```bash
npx wrangler secret put TG_WEBHOOK_SECRET
npx wrangler deploy
```

### 机器人命令

| 命令 | 功能 |
|---|---|
| `/start` `/menu` | 调出主菜单（内联键盘） |
| `/nodes` | 查看全部 VPS 的节点矩阵（协议:端口 / 状态 / 归属） |
| `/proxy <ip或名称> on\|off` | 远程开关住宅代理（不带 on/off 显示当前状态+按钮） |
| `/stats` | 近 7 天流量统计（按 VPS、按节点） |
| `/deploy8 <ip或名称> [起始端口]` | 一键下发 8 合 1 节点（XTLS-Reality / Hysteria2 / TUIC / Trojan / H2-Reality / gRPC-Reality / AnyTLS / Naive），带确认按钮 |
| `/set_interval 10` | 设置探针上报间隔（秒） |
| `/set_sitetitle 新标题` | 更改大盘标题 |

主菜单内联键盘还提供：探针节点列表、节点矩阵、住宅代理开关、流量统计、系统设置快捷开关。

> 安全：Webhook 同时校验 `X-Telegram-Bot-Api-Secret-Token` 与 `chat_id`，只有你本人能控制。

## 架构

```text
浏览器 / VPS Agent
        |
Cloudflare Worker
  |- Worker Assets: 前端与 VPS 安装文件
  |- /api/*: XUI 后端接口
  |- /agent/ws、/dashboard/*、/public/ws：内置实时服务
  |- Cron: 离线检查
  |- D1 (DB): 配置、用户、节点、流量、探针数据
  `- Durable Objects: VPS 实时状态与 Dashboard Hub
```

## 注意事项

- 一键部署默认使用 `admin/admin`；住宅代理默认关闭（凭据为空）。公开使用前必须将 `ADMIN_PASSWORD` 覆盖为强 Secret；启用住宅代理时再设置 `PROXY_USER`、`PROXY_PASS` 为独立强 Secret。
- 不要提交自定义 `ADMIN_PASSWORD`、D1 ID、Telegram Token 或代理凭据。
- `DB` 是固定 binding 名称，修改会导致后端无法访问数据库。
- 修改 Worker Variables 或 Bindings 后需要重新部署。
- 使用已有 D1 时，确认 `DB` 绑定指向正确数据库。
- `workspace-preview.html` 仅用于本地预览，不参与 Worker 静态资源发布。

## 开源协议

MIT
