# 轻松牙医 WorkBuddy 技能包

本目录可直接上传到 GitHub，供 WorkBuddy 安装。Skills 负责“怎么查、怎么确认”，MCP 负责“能不能查和写”。

## 安装步骤

### 1. 在轻松牙医后台生成 API Key

1. 登录轻松牙医管理后台。
2. 进入 **系统设置 → 诊所设置 → Agent 接入**。
3. 点击 **生成 WorkBuddy API Key**。
4. 复制弹窗中的 Key（明文只显示一次），妥善保存。该 Key 具备查询、快速建档、创建预约和医生排班写入能力。

如需使用运营日报工具，请由管理员在后端创建日报专用 Key：

```bash
cd backend
uv run mcp-manage create-client --tenant-id <租户ID> --name "Daily Reporter" --profile daily_reporter
```

日报专用 Key 具备预约、收费、病历、回访等只读查询能力，但不具备快速建档、创建预约和医生排班写入能力。一个 MCP 连接通常只配置一个 Key；需要同时使用业务写入和完整日报时，请按当前任务切换对应 Key。

### 2. 安装 Skills

将本目录安装到 WorkBuddy Skills 目录，例如：

```text
~/.workbuddy/skills/easydental-clinic/
```

常见方式：

- 从 GitHub 导入你自行维护的技能仓库；
- 或手动复制整个 `workbuddy-skill/` 文件夹到上述目录。

安装后重启 WorkBuddy。

### 3. 配置 MCP 连接器

复制 `mcp.json.example` 的内容到 WorkBuddy MCP 配置：

- 用户级：`~/.workbuddy/mcp.json`
- 或项目级：`<项目目录>/.workbuddy/mcp.json`

将示例中的：

- `192.168.1.100` 替换为运行 MCP Server 的机器局域网 IP；
- `在此粘贴API_Key` 替换为步骤 1 生成的 Key。

### 4. 启动 MCP Server

在 easydental 后端目录执行：

```bash
cd backend
uv sync --extra mcp
uv run upgrade
uv run mcp-server
```

默认监听 `http://127.0.0.1:8801`。

### 5. 验证

1. 在 WorkBuddy 插件面板查看 MCP 连接状态为绿色。
2. 新建任务时选择 `@easydental-clinic` 技能。
3. 尝试提问：“帮我查一下手机号 138 开头的患者”。
4. 创建患者、预约或写入排班时，确认 WorkBuddy 会先复述信息，并在你确认后再写入系统。

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `SKILL.md` | WorkBuddy 技能主文件 |
| `mcp.json.example` | MCP 连接器模板 |
| `reference.md` | 工具清单与示例对话 |

## 常见问题

**MCP 连接失败**

- 确认 MCP Server 已启动，且 WorkBuddy 能访问对应 IP 和端口。
- 确认 API Key 未过期、未被停用。
- 若重新生成过 Key，需同步更新 `mcp.json`。

**Skills 已安装但 Agent 不会查数据**

- 确认 MCP 连接器状态正常。
- 新建任务时手动指定 `@easydental-clinic`。

**Agent 创建预约前没有确认**

- 检查是否安装了最新 `SKILL.md`。
- MCP 写工具要求 `user_confirmed=true`，未确认时会返回中文错误，不会写入业务数据。

**Agent 不能写入排班**

- 确认使用的是 WorkBuddy 预约助手 Key，而不是 `agent_daily_reporter` 日报专用 Key。
- 确认已安装最新 `SKILL.md`，并先用 `list_schedule_shifts` 查询班次 ID。
- 清空排班时必须明确复述医生和日期，确认后调用写入工具且 `shift_id` 为空。

**Key 丢失**

- 后台 **Agent 接入** 页无法再次查看历史 Key，只能 **重新生成 Key** 并更新 WorkBuddy 配置。
