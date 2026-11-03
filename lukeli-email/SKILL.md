# SKILL：配置「邮箱连接中心」并验证转发链路

> 适用：用户在行程资料页点「进入邮箱连接中心」失败，或想从零把邮箱接入跑通。
> 你（Agent）应该有本机终端权限；本文件就是你的作业单，按顺序做，每一步向用户报告结果。

## 背景知识（先读再动手）

- 主应用：`http://localhost:8796`（shared 档，数据在仓库 `storage/`）；连接中心：`http://localhost:8790`。
- 一键进入 = 主应用签发 `…/#h=<token>` 一次性凭证，连接中心本地校验（两端共享 `~/.config/lukeli/handoff.env` 的密钥）。
- 真实邮件链路：用户邮箱自动转发 → Cloudflare Email Routing（chinatrips.us）→ Worker → 隧道 → 连接器 `/inbound/email`。下游 4xx/5xx 时 Gmail 约 6 分钟自动重试，「没立刻入库」≠「链路坏了」。
- 仓库根目录 = 包含 `scripts/svc.mjs` 的目录；也可以从 `curl -s localhost:8796/v1/health` 的 `storage` 字段去掉末尾 `/storage` 推出。

## 步骤

1. **体检（只读）**
   ```bash
   node scripts/svc.mjs status
   curl -s localhost:8796/v1/health    # 期望 {"ok":true,...,"storage":"<仓库>/storage"}
   curl -s localhost:8790/health       # 期望 {"ok":true,"mock":...,"tenants":...}
   ```
2. **起服务**（缺哪个起哪个；shared 档重启前必须征得用户同意）
   ```bash
   node scripts/svc.mjs up app --shared --i-know
   node scripts/svc.mjs up connector --shared --i-know
   ```
   连接器探活失败（`connector-unreachable`）就是没起，不要改配置硬绕。
3. **一键进入自检**：让用户登录 `http://localhost:8796`（AUTH_DEV 下会给可点链接），进任意行程「资料」页点「进入邮箱连接中心」。
   - 出现「连接中心没在运行（试过 …）」→ 回到第 2 步。
   - 出现「handoff-not-configured」→ `node scripts/handoff-secret.mjs` 生成密钥后**重启两端**。
   - 出现 `error:"origin"` → 检查 APP_ORIGIN 是否被写成 `0.0.0.0:port`（应一律 `http://localhost:<port>`）。
4. **真实邮件验收（可选，需要 Cloudflare token 就绪）**：让用户从自己邮箱转发一封到 `t+<token>@chinatrips.us`，5 分钟内连接中心应出现该邮件；不行先看连接器审计与 `index.jsonl`，别急着重发。

## 红线

- 不要向用户索取任何明文密码 / 授权码；IMAP 授权码只报位数。
- 不要 kill 用户正在使用的进程，除非明确说明并征得同意。
- 不要把密钥、token 回显到聊天里。

## 完成的定义

- `/health` 两端形状正确；资料页一键进入直达连接中心；（可选）一封真实转发邮件入索引。
