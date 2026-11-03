# SKILL：接入「微信聊天」（WeChatBridge / 微信流）

> 目标：让用户在微信里「多选 → 转发」就把聊天送进路客理，全程不导出文件。
> 你（Agent）有本机终端权限；按顺序做，每步向用户报告结果。

## 背景知识

- 集成对象：[WeChatBridge（微信流）](https://github.com/freestylefly/WeChatBridge)。macOS 微信 4.1.13+。
- 它收到的聊天包落在共享容器 `Inbox/Ready`，路客理监听这些目录并作为「候选」展示：
  - `~/Library/Application Support/WeChatBridge/Inbox/Ready`
  - `~/Library/Group Containers/DLKMC3ZRZQ.com.xiangming.wechatbridge.shared/Inbox/Ready`
- 兜底通道（不装也能用）：把导出文件放进 `~/Downloads/路客理微信收件箱`（可用 env `TRIP_WECHAT_INBOX` 覆盖）。
- 红线：**不读取微信数据库，不解密、不注入微信进程**；图片和语音不进复盘。

## 步骤

1. **检测是否已装**
   ```bash
   ls ~/Library/Application\ Support/WeChatBridge/Inbox/Ready 2>/dev/null && echo 已安装
   ```
2. **未装 → 安装**（发行在蓝奏云，有提取码，无法脚本化下载，引导用户手动）：
   - 下载页 `https://wwbsr.lanzout.com/b0umjur8h`，提取码 **向明**，取 macOS dmg；
   - 把「微信流」拖进「应用程序」并打开一次；系统询问权限（辅助功能/屏幕录制）时允许；
   - 确认用户微信是 macOS 4.1.13+。
3. **验证服务在跑**
   ```bash
   curl -s localhost:8796/v1/health   # 期望 {"ok":true,...}
   ```
   没起：`node scripts/svc.mjs up app --shared --i-know`（shared 重启前征得用户同意）。
4. **让用户试转发**：微信里多选几条消息 → 合并转发 → 「转发到其他应用」→ 选「微信流」的入口。
5. **验证**：行程资料页「微信聊天」卡第一行变为「最省事：…」；候选区几秒内出现刚转发的包（来源标「微信流」），点一下即接入本旅程。
   - 候选没出现：确认 `Inbox/Ready` 下有新批次目录；再查资料页是否打开了（收件箱监听随服务端自转，不需要人工扫盘）。

## 完成的定义

- 资料页微信卡显示转发就绪；一次真实转发的包被接入并计入消息数。
