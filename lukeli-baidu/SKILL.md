# SKILL：接入「百度网盘」（手动配置参考）

> 注意：百度网盘在资料页是**一键扫码授权**，正常情况下用户自己点「扫码 / 打开百度授权」即可，本 SKILL 供授权失败或应用未配置时使用。

## 背景知识

- OAuth 应用配置：`data/cloud/baidu.json`（`appKey` / `secret` / 可选 `redirect`）。
- 回调地址必须与百度控制台**逐字一致**：`BAIDU_REDIRECT` env > 配置里的 `redirect` > `<APP_ORIGIN>/v1/connectors/baidu/callback`。
- 授权按「槽位」存储：`storage/tenants/<tenantId>/connectors/baidu/<slot>.json`；`default` = 租户默认账号，`trip:<tripId>` = 该旅程专用。读旅程先找专槽、没有回退默认（= 新旅程自动复用旧授权）；旧版单文件 `baidu.json` 首读自动迁移。
- 红线：不要向用户索取任何 cookie / 密码；token 文件已加密（AES-GCM），不要尝试读明文。

## 步骤

1. **检查应用配置**
   ```bash
   cat data/cloud/baidu.json   # 需要 appKey 与 secret
   ```
   没有：引导用户到百度网盘开放平台创建应用，回调地址填 `<APP_ORIGIN>/v1/connectors/baidu/callback`，写入该文件后重启主应用。
2. **起服务**
   ```bash
   node scripts/svc.mjs up app --shared --i-know   # shared 重启前征得用户同意
   ```
3. **授权**：用户在行程资料页点「扫码 / 打开百度授权」，用百度 App 扫码确认。回调成功会带上账号名（uinfo 的 baidu_name）。
4. **验证**
   - 资料页显示「已连接 · 账号 xxx · 沿用默认账号 / 本旅程专用账号」；
   - 目录浏览可列出条目，「同步当前目录」能把照片拉进旅程。

## 常见失败

| 现象 | 出路 |
|---|---|
| `redirect_uri_mismatch` | 授权跳转与 token 交换的 redirect 必须逐字一致；对齐控制台登记值 |
| `baidu-not-configured` | 缺 `data/cloud/baidu.json` |
| 授权成功但列表 409 | 该旅程槽位与默认槽位都为空；从资料页重新授权一次 |
| 换账号影响到了别的旅程 | 槽位写错了：确认是从旅程页发起的授权（会写 `trip:<tripId>`） |
