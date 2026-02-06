# 飞书交互式卡片技能

## 快速开始

### 1. 安装依赖

```bash
cd scripts
npm install
```

### 2. 启动回调服务器

```bash
node card-callback-server.js
```

### 3. 发送测试卡片

```bash
# 确认卡片
node send-card.js confirmation "确认删除文件？" --chat-id oc_xxx

# TODO 清单
node send-card.js todo --chat-id oc_xxx

# 投票
node send-card.js poll "本周团建活动" --options "保龄球,看电影,聚餐" --chat-id oc_xxx

# 表单
node send-card.js form --chat-id oc_xxx
```

## 目录结构

```
feishu-interactive-cards/
├── SKILL.md                          # 技能文档
├── README.md                         # 本文件
├── scripts/
│   ├── card-callback-server.js       # 回调服务器
│   ├── send-card.js                  # 发送卡片脚本
│   ├── card-templates.js             # 卡片模板库
│   └── package.json                  # 依赖配置
└── examples/
    ├── confirmation-card.json        # 确认卡片示例
    ├── todo-card.json                # 任务清单示例
    ├── poll-card.json                # 投票卡片示例
    └── form-card.json                # 表单卡片示例
```

## 在 Agent 中使用

```javascript
// 发送确认卡片
await exec({
  command: `node E:\\openclaw\\workspace\\skills\\feishu-interactive-cards\\scripts\\send-card.js confirmation "确认删除文件 test.txt？" --chat-id ${chatId}`
});

// 发送 TODO 清单
await exec({
  command: `node E:\\openclaw\\workspace\\skills\\feishu-interactive-cards\\scripts\\send-card.js todo --chat-id ${chatId}`
});
```

## 配置

确保 `~/.openclaw/openclaw.json` 包含飞书配置：

```json
{
  "channels": {
    "feishu": {
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        }
      }
    }
  },
  "gateway": {
    "enabled": true,
    "port": 18789,
    "token": "your-gateway-token"
  }
}
```

## 更多信息

详见 [SKILL.md](SKILL.md)
