---
name: feishu-interactive-cards
description: 飞书交互式卡片增强技能 - 当需要向飞书发送消息时，优先使用交互式卡片而非纯文本，支持按钮、表单、投票等丰富交互。自动启动回调服务器接收用户交互。
when: "当 agent 需要向飞书发送任何消息时，特别是包含选择、确认、表单填写等交互场景"
examples:
  - "发送一个任务清单到飞书"
  - "在飞书中创建一个投票"
  - "向飞书发送需要确认的消息"
  - "发送问卷调查到飞书"
  - "在飞书中展示待办事项"
metadata:
  openclaw:
    requires:
      bins: ["node"]
    emoji: "🎴"
    primaryEnv: null
---

# 飞书交互式卡片增强技能

## 概述

这个技能让 OpenClaw Agent 在向飞书发送消息时，**优先使用交互式卡片**而非纯文本。交互式卡片提供了丰富的用户界面，包括按钮、表单、投票等，让用户可以直接在卡片上进行操作，而不需要输入文字回复。

## 核心原则

### 🎯 何时使用交互式卡片

**必须使用交互式卡片的场景：**
- ✅ 需要用户做出选择（是/否、多选一）
- ✅ 需要用户确认操作
- ✅ 展示任务清单、待办事项
- ✅ 发起投票或问卷调查
- ✅ 需要用户填写表单信息
- ✅ 展示需要交互的数据（如可切换状态的列表）
- ✅ 任何不确定用户意图的情况

**可以使用纯文本的场景：**
- ❌ 简单的信息通知（无需回复）
- ❌ 纯粹的数据展示（无需交互）
- ❌ 已经明确的指令执行结果

### 🚫 重要约束

**当有任何不确定的情况时：**
1. **不要直接发送文字消息**
2. **创建交互式卡片**，让用户通过按钮选择
3. **等待用户交互**，然后根据用户选择执行相应操作

**示例：**
- ❌ 错误："我帮你删除这个文件了"（直接执行）
- ✅ 正确：发送卡片 "确认删除文件？" [确认] [取消]

## 技能结构

```
feishu-interactive-cards/
├── SKILL.md                          # 本文件
├── scripts/
│   ├── card-callback-server.js       # 回调服务器（长连接模式）
│   ├── send-card.js                  # 发送卡片的通用脚本
│   ├── card-templates.js             # 卡片模板库
│   └── package.json                  # Node.js 依赖
└── examples/
    ├── confirmation-card.json        # 确认卡片示例
    ├── todo-card.json                # 任务清单示例
    ├── poll-card.json                # 投票卡片示例
    └── form-card.json                # 表单卡片示例
```

## 使用方法

### 1. 启动回调服务器

回调服务器使用**长连接模式**，无需配置公网域名或回调地址。

```bash
cd E:\openclaw\workspace\skills\feishu-interactive-cards\scripts
node card-callback-server.js
```

**特点：**
- ✅ 使用飞书长连接，无需公网 IP
- ✅ 自动重连，稳定可靠
- ✅ 实时接收卡片交互回调
- ✅ 自动发送回调数据到 OpenClaw Gateway

**配置要求：**
- 飞书开发者后台选择"长连接"模式
- 订阅 `card.action.trigger` 事件
- 配置正确的 App ID 和 App Secret

### 2. 发送交互式卡片

使用 `send-card.js` 脚本发送卡片：

```bash
# 发送确认卡片
node scripts/send-card.js confirmation "确认删除文件？" --chat-id oc_xxx

# 发送任务清单
node scripts/send-card.js todo --chat-id oc_xxx

# 发送投票
node scripts/send-card.js poll "本周团建活动" --options "保龄球,看电影,聚餐" --chat-id oc_xxx

# 发送自定义卡片
node scripts/send-card.js custom --template examples/custom-card.json --chat-id oc_xxx
```

### 3. 在 Agent 中使用

当 Agent 需要向飞书发送消息时，应该：

```javascript
// ❌ 错误做法：直接发送文字
await message({ 
  action: "send", 
  channel: "feishu", 
  message: "是否确认删除？" 
});

// ✅ 正确做法：发送交互式卡片
await exec({
  command: `node E:\\openclaw\\workspace\\skills\\feishu-interactive-cards\\scripts\\send-card.js confirmation "确认删除文件 test.txt？" --chat-id ${chatId}`
});
```

## 卡片模板

### 1. 确认卡片（Confirmation Card）

用于需要用户确认的操作。

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "content": "⚠️ 确认操作", "tag": "plain_text" },
    "template": "orange"
  },
  "elements": [
    {
      "tag": "div",
      "text": {
        "content": "**确认删除文件 test.txt？**\n\n此操作不可撤销。",
        "tag": "lark_md"
      }
    },
    {
      "tag": "hr"
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "✅ 确认", "tag": "plain_text" },
          "type": "danger",
          "value": { "action": "confirm", "operation": "delete_file", "file": "test.txt" }
        },
        {
          "tag": "button",
          "text": { "content": "❌ 取消", "tag": "plain_text" },
          "type": "default",
          "value": { "action": "cancel" }
        }
      ]
    }
  ]
}
```

### 2. 任务清单卡片（TODO Card）

展示可交互的任务列表。

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "content": "📋 今日任务清单", "tag": "plain_text" },
    "template": "blue"
  },
  "elements": [
    {
      "tag": "div",
      "text": {
        "content": "**今日任务** (0/3 已完成)",
        "tag": "lark_md"
      }
    },
    {
      "tag": "hr"
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "⬜ 完成项目报告", "tag": "plain_text" },
          "type": "primary",
          "value": { "action": "toggle_todo", "todoId": "todo1" }
        }
      ]
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "⬜ 回复客户邮件", "tag": "plain_text" },
          "type": "primary",
          "value": { "action": "toggle_todo", "todoId": "todo2" }
        }
      ]
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "⬜ 准备周会材料", "tag": "plain_text" },
          "type": "primary",
          "value": { "action": "toggle_todo", "todoId": "todo3" }
        }
      ]
    }
  ]
}
```

### 3. 投票卡片（Poll Card）

创建投票或问卷。

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "content": "📊 团队投票", "tag": "plain_text" },
    "template": "purple"
  },
  "elements": [
    {
      "tag": "div",
      "text": {
        "content": "**本周团建活动选择**\n\n请投票选择你最想参加的活动：",
        "tag": "lark_md"
      }
    },
    {
      "tag": "hr"
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "🎳 保龄球", "tag": "plain_text" },
          "type": "default",
          "value": { "action": "vote", "option": "bowling" }
        },
        {
          "tag": "button",
          "text": { "content": "🎬 看电影", "tag": "plain_text" },
          "type": "default",
          "value": { "action": "vote", "option": "movie" }
        },
        {
          "tag": "button",
          "text": { "content": "🍕 聚餐", "tag": "plain_text" },
          "type": "default",
          "value": { "action": "vote", "option": "dinner" }
        }
      ]
    }
  ]
}
```

### 4. 表单卡片（Form Card）

收集用户输入信息。

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "content": "📝 信息收集", "tag": "plain_text" },
    "template": "blue"
  },
  "elements": [
    {
      "tag": "form",
      "name": "user_form",
      "elements": [
        {
          "tag": "div",
          "text": { "content": "**姓名**", "tag": "lark_md" }
        },
        {
          "tag": "input",
          "name": "name",
          "placeholder": { "content": "请输入姓名", "tag": "plain_text" }
        },
        {
          "tag": "div",
          "text": { "content": "**邮箱**", "tag": "lark_md" }
        },
        {
          "tag": "input",
          "name": "email",
          "placeholder": { "content": "请输入邮箱", "tag": "plain_text" }
        }
      ]
    },
    {
      "tag": "hr"
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": { "content": "✅ 提交", "tag": "plain_text" },
          "type": "primary",
          "value": { "action": "submit_form" }
        }
      ]
    }
  ]
}
```

## 回调处理

### Gateway 集成

回调服务器会自动将所有卡片交互发送到 OpenClaw Gateway：

```
POST http://localhost:18789/api/callback
Authorization: Bearer {GATEWAY_TOKEN}
Content-Type: application/json

{
  "type": "feishu_card_callback",
  "timestamp": "2026-02-06T10:30:00.000Z",
  "data": {
    "event_id": "...",
    "operator": { "open_id": "...", "user_id": "..." },
    "action": { "value": { "action": "confirm", "operation": "delete_file" } },
    "context": { "open_message_id": "...", "open_chat_id": "..." }
  }
}
```

### 在 Agent 中处理回调

Agent 可以通过监听 Gateway 的回调来响应用户交互：

```javascript
// 示例：处理确认操作
if (callback.data.action.value.action === "confirm") {
  const operation = callback.data.action.value.operation;
  const file = callback.data.action.value.file;
  
  // 执行实际操作
  await exec({ command: `rm ${file}` });
  
  // 更新卡片显示结果
  await updateCard(callback.context.open_message_id, {
    header: { title: "✅ 操作完成", template: "green" },
    elements: [
      { tag: "div", text: { content: `文件 ${file} 已删除`, tag: "lark_md" } }
    ]
  });
}
```

## 最佳实践

### 1. 卡片设计原则

- **清晰明确**：卡片标题和内容要清楚说明目的
- **操作明显**：按钮文字要明确表达操作结果
- **视觉层次**：使用分隔线、标题、颜色区分不同部分
- **防误操作**：危险操作使用 `danger` 类型按钮

### 2. 交互流程

```
用户请求 → Agent 判断 → 发送交互式卡片 → 用户点击按钮 
→ 回调服务器接收 → 发送到 Gateway → Agent 处理 → 更新卡片/执行操作
```

### 3. 错误处理

- **超时处理**：如果用户长时间未响应，可以发送提醒
- **重复点击**：回调服务器已内置去重机制（3秒内重复请求会被忽略）
- **异常恢复**：如果操作失败，更新卡片显示错误信息

### 4. 性能优化

- **卡片状态**：在按钮的 `value` 中携带完整状态，避免额外查询
- **异步处理**：回调处理应该快速响应，耗时操作放到后台
- **批量操作**：多个相关操作可以合并到一张卡片中

## 配置

### 飞书应用配置

在 `~/.openclaw/openclaw.json` 中配置：

```json
{
  "channels": {
    "feishu": {
      "accounts": {
        "main": {
          "appId": "YOUR_APP_ID",
          "appSecret": "YOUR_APP_SECRET"
        }
      }
    }
  },
  "gateway": {
    "enabled": true,
    "port": 18789,
    "token": "YOUR_GATEWAY_TOKEN"
  }
}
```

### 回调服务器配置

回调服务器会自动从 `openclaw.json` 读取配置，无需额外设置。

## 故障排查

### 问题：点击按钮没有反应

**可能原因：**
1. 回调服务器未启动
2. 飞书后台未选择"长连接"模式
3. 未订阅 `card.action.trigger` 事件

**解决方案：**
```bash
# 检查回调服务器是否运行
ps aux | grep card-callback-server

# 重启回调服务器
cd E:\openclaw\workspace\skills\feishu-interactive-cards\scripts
node card-callback-server.js
```

### 问题：Gateway 未收到回调

**可能原因：**
1. Gateway 未启动
2. Gateway Token 配置错误

**解决方案：**
```bash
# 启动 Gateway
E:\openclaw\workspace\scripts\gateway.cmd

# 检查配置
type %USERPROFILE%\.openclaw\openclaw.json
```

### 问题：卡片显示异常

**可能原因：**
1. JSON 格式错误
2. 必填字段缺失
3. 字段类型不匹配

**解决方案：**
- 使用提供的模板作为基础
- 参考飞书官方文档验证字段
- 使用 JSON 验证工具检查格式

## 参考资源

- [飞书开放平台 - 消息卡片](https://open.feishu.cn/document/ukTMukTMukTM/uczM3QjL3MzN04yNzcDN)
- [飞书开放平台 - 卡片交互](https://open.feishu.cn/document/ukTMukTMukTM/uYjNwUjL2YDM14iN2ATN)
- [OpenClaw 文档](https://docs.openclaw.ai)

## 更新日志

- **2026-02-06**: 初始版本
  - 支持确认、任务清单、投票、表单等常见卡片类型
  - 集成 Gateway 回调处理
  - 使用长连接模式，无需公网域名
