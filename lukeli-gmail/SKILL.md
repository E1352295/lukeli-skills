# SKILL：接入「Gmail」（注册 Google OAuth 客户端 + 配置连接器）

> 目标：把连接中心的 Gmail 从演示连接变成真实 Google 授权。
> 你（Agent）可以使用浏览器操控（用户已登录自己的 Google 账号），或引导用户手动点击。
> 红线：不接触用户的 Google 密码；Client Secret 拿到后只写入本地配置文件，不回显到聊天。

## 背景知识

- 连接器已实现完整 Gmail OAuth + Gmail API 拉信（`connector/server.mjs`），缺的只是 OAuth 客户端凭证。
- 回调地址（必须与 Google 控制台逐字一致）：
  - 本机：`http://localhost:8790/oauth/google/callback`
- 凭证存放：`~/.config/lukeli/google.env`（格式见下），连接器启动时由 svc 注入。
- 测试模式即可：自用不需要通过 Google 应用验证，把用户自己的 Gmail 加为测试用户就行。

## 步骤

1. **确认服务在跑**
   ```bash
   curl -s localhost:8790/health      # 期望 {"ok":true,...}
   ```
2. **创建 OAuth 客户端**（用浏览器操控或引导用户）：
   - 打开 `https://console.cloud.google.com/projectcreate` 建项目（名字随意，如 `lukeli`）；
   - 「API 和服务 → OAuth 同意屏幕」：External → 填应用名与用户邮箱 → 测试用户加用户自己的 Gmail；
   - 「凭据 → 创建凭据 → OAuth 客户端 ID」：类型 Web 应用；已获授权的重定向 URI 填上面的回调地址；
   - 拿到 **Client ID** 与 **Client Secret**。
3. **写入本地配置**（不要回显 Secret 全文）：
   ```bash
   cat > ~/.config/lukeli/google.env <<EOF
   GOOGLE_CLIENT_ID=<client_id>
   GOOGLE_CLIENT_SECRET=<client_secret>
   EOF
   chmod 600 ~/.config/lukeli/google.env
   ```
   svc 的连接器启动时会自动读取该文件并注入环境变量；改完需要重启连接器：
   `node scripts/svc.mjs down connector --shared --i-know && node scripts/svc.mjs up connector --shared --i-know`（shared 重启前征得用户同意）。
4. **验证**：连接中心 Gmail 卡点「一键连接 Gmail」→ Google 授权页（测试模式提示点「继续」）→ 回到连接中心显示已连接 → 收件箱出现真实 Gmail 邮件。

## 常见失败

| 现象 | 出路 |
|---|---|
| `redirect_uri_mismatch` | 控制台回调地址与连接器配置必须逐字一致 |
| 授权页提示「应用未通过验证」 | 测试模式正常现象；确认该 Gmail 已加为测试用户 |
| 连接后仍显示演示 | 连接器没重启或 google.env 未被读取；检查启动日志是否注入了 GOOGLE_CLIENT_ID |
