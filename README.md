# 京东签到 (GitHub Actions)

零本地资源占用：不用装青龙面板，不用 NAS 常驻进程。脚本由 GitHub 免费提供的 Actions 云端按计划自动执行，跑完自动推送结果。

## 特点

- **本地零占用** — 不需要 Docker、不需要常驻面板
- **多账号** — 支持 `JD_COOKIE` / `JD_COOKIE2` / `JD_COOKIE3`（想加更多可自行在 `jd_sign.js` 和 workflow 里扩展）
- **多通道推送** — Server酱（微信）、Bark（iOS）
- **自动定时** — 每天北京时间 7:15 执行（错峰避开凌晨高峰），也可手动触发
- **风控优化** — 请求间自动加 5-15 秒随机延迟，默认关闭超市/秒杀/现金红包等易风控接口

## 快速开始

### 1. 把仓库推到 GitHub

用你的 GitHub 账号新建一个仓库（Public/Private 都可以），把本目录内容推上去：

```bash
cd jd-actions
git init
git add .
git commit -m "init: jd sign"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

### 2. 配置 Secrets

到仓库 `Settings` → `Secrets and variables` → `Actions` → `New repository secret`，逐个添加：

| 变量名 | 必填 | 说明 |
|---|---|---|
| `JD_COOKIE` | ✅ | 京东账号 Cookie（注意是 `pt_key=xxx;pt_pin=yyy;` 格式） |
| `JD_COOKIE2` | 选填 | 第二个账号 |
| `JD_COOKIE3` | 选填 | 第三个账号 |
| `PUSH_KEY` | 选填 | Server酱 SendKey（微信推送） |
| `BARK_KEY` | 选填 | Bark 推送 key（iOS） |

> `PUSH_KEY`、`BARK_KEY` 至少要配一个，否则跑完没有通知，只能去 Actions 页面看日志。

### 3. 获取京东 Cookie

1. 用 Chrome 打开 https://bean.m.jd.com/ 并登录
2. F12 打开开发者工具 → `Network`（网络）面板
3. 刷新页面，随便点一个请求，找到 `Cookie` 请求头
4. 复制其中的 `pt_key=...;pt_pin=...;` 这一段（一般包含 `pt_key` 和 `pt_pin` 两个字段）

> Cookie 有效期约 **1 个月**，过期后签到会显示「Cookie 失效」，届时重新抓取并更新 Secrets 即可。

### 4. 配置推送

**Server酱（微信）：**
- 登录 https://sct.ftqq.com/ ，授权 GitHub，微信扫码绑定
- 找到「SendKey」，复制填入 `PUSH_KEY`

**Bark（iOS）：**
- App Store 安装 Bark，授权后拿到一个 URL，最后的 key 部分填入 `BARK_KEY`

### 5. 测试

进入仓库 `Actions` 页面 → 左侧选中 `JD Sign` → 点 `Run workflow` 手动触发一次。等 1-2 分钟看执行日志和推送结果。

## 说明

- 定时触发有几分钟延迟属正常；GitHub 免费额度每月 2000 分钟，签到每天跑几分钟，完全够用。
- 想改执行时间：编辑 `.github/workflows/jd.yml` 里的 `cron`，注意是 **UTC 时间**（北京时间 = UTC + 8 小时）。
- 领取的京豆数量不稳定，且部分接口限量，属于正常现象。
- **风险提示**：脚本通过逆向京东接口领取京豆，有违反《京东用户协议》的可能，存在账号被风控的风险，请自行评估后使用。

## 默认执行时间参考

| 北京时间 | 对应 cron (UTC) |
|---|---|
| 每天 00:00 | `0 16 * * *`（当前用 00:15 = `15 16 * * *`） |

## 依赖

- Node.js（GitHub Actions 环境自动安装）
- `request` 模块（NobyDa 脚本内部使用）
