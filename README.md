# lukeli-skills

路客理（Luke Li 旅程复盘）的 connector 接入 SKILL。
用户在资料页点「复制提示词」后，把这段话发给任意 Agent；Agent 读取对应 SKILL 即可完成配置与验证。

| SKILL | 用途 |
|---|---|
| [lukeli-baidu](./lukeli-baidu/SKILL.md) | 百度网盘（授权失败/未配置时排障） |
| [lukeli-email](./lukeli-email/SKILL.md) | 邮箱连接中心与转发链路 |
| [lukeli-gmail](./lukeli-gmail/SKILL.md) | 百度网盘（授权失败/未配置时排障） |
| [lukeli-wechat](./lukeli-wechat/SKILL.md) | 微信聊天（WeChatBridge / 微信流） |

原则：Agent 只做配置与验证，不向用户索取任何明文密码/授权码。
