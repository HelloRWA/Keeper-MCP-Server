# Keeper MCP Server

**Daily Keep** 内置 MCP（Model Context Protocol）服务，让 AI 助手（Claude、Cursor、任意 MCP 客户端）直接操作你的习惯打卡、待办事项和计时器，无需打开浏览器。

---

## 快速接入

### 1. 获取 API Token

登录 DK → 右上角设置 → **API Token 管理** → 创建 Token。

Token 格式：`dk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`，创建后只显示一次，请立即复制保存。

### 2. 配置 MCP 客户端

**Endpoint**

```
https://dk.jjf-tech.cn/api/mcp
```

**协议**：MCP Streamable HTTP（JSON-RPC 2.0 over HTTP POST）

**认证**：Bearer Token

```
Authorization: Bearer dk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### Claude Desktop 配置示例

编辑 `~/Library/Application Support/Claude/claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "daily-keep": {
      "url": "https://dk.jjf-tech.cn/api/mcp",
      "headers": {
        "Authorization": "Bearer dk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

### Cursor 配置示例

在 Cursor 设置 → MCP → 添加服务器：

```json
{
  "name": "daily-keep",
  "url": "https://dk.jjf-tech.cn/api/mcp",
  "headers": {
    "Authorization": "Bearer dk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

---

## 可用工具

### 习惯打卡

| 工具 | 说明 |
|------|------|
| `list_habits` | 列出所有习惯，含今日打卡状态和累计天数 |
| `get_habit_detail` | 查看某个习惯详情：连续天数、最近 7 天记录 |
| `checkin_habit` | 今日打卡，奖励 2 努力值 |
| `makeup_checkin` | 补打过去 1–10 天的卡，消耗 5 努力值 |

**示例对话**

> "帮我打卡今天的运动习惯"
> "我上周三忘记打卡了，帮我补一下"
> "我的习惯今天都打了吗？"

---

### 待办事项（Todo）

| 工具 | 说明 |
|------|------|
| `list_todos` | 列出某习惯下所有待办，含子任务层级 |
| `add_todo` | 添加待办，可设置计时器时长和父任务 |
| `toggle_todo` | 切换完成 / 未完成状态 |
| `delete_todo` | 删除待办及其所有子任务 |

**示例对话**

> "给我的写作习惯加一个 Todo：整理大纲，25 分钟"
> "把读书列表里的《原则》标记为已完成"
> "今天的待办还有哪些没做？"

---

### 计时器

| 工具 | 说明 |
|------|------|
| `start_countdown` | 开始倒计时，记录开始时间戳 |
| `stop_countdown` | 停止/暂停，累计已用时间并更新 `[⏱Xm]` 标记 |

计时记录同步到 DK 网页端，提交流水账时会统计总耗时。

**示例对话**

> "开始计时，我要专注写代码了"
> "暂停，我去接个电话"
> "停止计时，今天写了多久？"

---

### 计次

| 工具 | 说明 |
|------|------|
| `increment_count` | 待办计次 +1，更新 `[×N]` 标记 |

适合记录重复性动作：俯卧撑组数、番茄钟轮数、阅读章节数等。

**示例对话**

> "俯卧撑又做了一组"
> "番茄钟完成一个，记一下"

---

### 流水账

| 工具 | 说明 |
|------|------|
| `submit_journal` | 将当前所有 Todo 提交为日记条目，清空已完成项，重置未完成项的时间和计次 |

**示例对话**

> "今天结束了，帮我提交流水账"
> "把今天的工作记录归档"

---

## 典型工作流

```
早上
  → list_habits           # 看今天有哪些习惯
  → checkin_habit         # 晨间习惯打卡
  → list_todos            # 规划今天的任务
  → add_todo              # 临时加任务

专注工作
  → start_countdown       # 开始番茄钟
  → stop_countdown        # 暂停/结束
  → increment_count       # 记录次数

晚上
  → toggle_todo           # 标记完成
  → submit_journal        # 提交今日流水账
```

---

## 技术规范

- **协议版本**：MCP `2025-03-26`
- **传输方式**：HTTP POST（Streamable HTTP）
- **Server Name**：`daily-keep-mcp`
- **支持方法**：`initialize` / `tools/list` / `tools/call` / `ping`
- **认证方式**：Session Cookie（网页端）或 Bearer Token（PAT，推荐用于 MCP）
- **Token 上限**：每个账号最多 10 个
- **Token 安全**：创建后仅展示一次，服务端仅存储原始值用于校验；每次调用自动更新 `last_used_at`
