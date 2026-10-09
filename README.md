# openclaw-qqbot (fork)

> 本仓库是 [tencent-connect/openclaw-qqbot](https://github.com/tencent-connect/openclaw-qqbot) 的 fork。原项目文档、说明、Issue 请访问原仓库。本 fork 只记录新增功能。

## 新增功能

基于原版 v2.0.3，在 `policy-injector` 中间件增加了群消息触发的可配置项，通过 OpenClaw 配置 `channels.qqbot.groups.<groupOpenid>` 启用。两个功能独立、可共存，任一命中即触发 AI 回复。

### replyUsers — per-user 免@触发白名单

名单内 openid 用户发的群消息（即使不@机器人）都触发 AI 回复（对其 requireMention 置 false），其余用户仍按群级 requireMention 判断。

```json
{
  "channels": {
    "qqbot": {
      "groups": {
        "<groupOpenid>": {
          "replyUsers": ["<openid1>", "<openid2>"]
        }
      }
    }
  }
}
```

### replyKeywords — 关键词触发

群消息（不@）content 包含任一关键词时触发 AI 回复（requireMention 置 false），不含关键词的不@消息不触发。群级配置，对所有成员生效。

```json
{
  "channels": {
    "qqbot": {
      "groups": {
        "<groupOpenid>": {
          "replyKeywords": ["为什么", "怎么", "什么是"]
        }
      }
    }
  }
}
```

### 触发逻辑

群消息进入 policy-injector 后：

1. sender openid 在 replyUsers → requireMention = false
2. content 含 replyKeywords 任一关键词 → requireMention = false
3. 否则 → requireMention = 群级配置（默认 true，需@才触发）

@机器人始终正常触发，不受影响。优先级：具体群 > 通配 "*" > 空。

## 前置条件

- QQ Bot 开通全量模式（GROUP_MESSAGE_CREATE 事件，intent GROUP_AND_C2C_EVENT 1<<25），否则收不到不@的群消息。
- 群级 requireMention 默认 true（defaultRequireMention）。

## 改动文件

- `src/types.ts` — GroupConfig 增加 replyUsers / replyKeywords 字段
- `src/config.ts` — DEFAULT_GROUP_CONFIG + resolveGroupConfigFromAccount 解析两字段
- `src/middleware/policy-injector.ts` — 群消息触发判断逻辑

## 构建

```bash
npm install
npm run build   # tsup → dist/index.cjs
```

产物 `dist/index.cjs` 覆盖到 OpenClaw 已安装插件目录的 `dist/`，重启 gateway 生效。
