# MCP 工具参考

## 可用工具

| 工具名 | 说明 |
| --- | --- |
| `health_check` | 检查 MCP Server 是否正常 |
| `search_patients` | 按关键词搜索患者（手机号脱敏） |
| `get_patient_detail` | 按患者 ID 查询详情 |
| `list_patient_appointments` | 查询患者预约列表 |
| `list_doctors` | 列出在职医生 |
| `list_doctor_schedule` | 查询医生某天排班与占用 |
| `find_available_slots` | 查询医生某天空闲时段 |

## 示例对话

### 查患者

用户：帮我找一下叫张三的患者

Agent 步骤：

1. 调用 `search_patients(keyword="张三")`
2. 若只有 1 条，可直接展示摘要
3. 若用户需要完整手机号，再调用 `get_patient_detail(patient_id=...)`

### 查预约

用户：看一下 patient-001 最近有哪些预约

Agent 步骤：

1. 调用 `list_patient_appointments(patient_id="patient-001")`
2. 按返回的 `summary` 和 `items` 组织回复

### 查空闲时段

用户：张医生明天还有哪些空档？

Agent 步骤：

1. 调用 `list_doctors()` 找到张医生的 `doctor_id`
2. 调用 `find_available_slots(doctor_id=..., schedule_date="2026-07-03")`
3. 列出 `available_slots`

## 认证方式

MCP Server 支持以下任一方式携带 API Key（命中其一即可）：

1. Query 参数：`?api_key=edmcp_xxx`
2. Header：`Authorization: Bearer edmcp_xxx`
3. Header：`X-EasyDental-Api-Key: edmcp_xxx`

内网 WorkBuddy 场景推荐使用 Query 参数，配置最简单。
