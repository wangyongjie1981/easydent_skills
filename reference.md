# MCP 工具参考

## 可用工具

| 工具名 | 说明 |
| --- | --- |
| `health_check` | 检查 MCP Server 是否正常 |
| `search_patients` | 按关键词搜索患者（手机号脱敏） |
| `get_patient_detail` | 按患者 ID 查询详情 |
| `list_patient_appointments` | 查询患者预约列表 |
| `get_current_clinic_profile` | 查询当前 API Key 绑定诊所 |
| `list_appointment_projects` | 查询可预约诊疗项目 |
| `quick_create_patient` | 按手机号查重并在确认后快速建档 |
| `create_appointment` | 在确认后创建普通预约 |
| `list_doctors` | 列出在职医生 |
| `list_schedule_shifts` | 列出启用班次，供写入排班前选择班次 ID |
| `list_doctor_schedule` | 查询医生某天排班与占用 |
| `batch_set_doctor_schedule` | 在确认后批量设置或清空医生排班 |
| `find_available_slots` | 查询医生某天空闲时段 |
| `get_daily_clinic_operations` | 查询指定日期诊所运营总览 |
| `list_daily_appointments` | 分页查询某日预约明细 |
| `list_daily_billing_records` | 分页查询某日收费、待收费或退款记录 |
| `get_daily_medical_record_report` | 查询某日病历状态统计和待补病历候选 |
| `list_daily_followup_items` | 分页查询某日回访待办、已回访、逾期、需医生处理或明日待办 |

> 运营日报工具需要使用 `agent_daily_reporter` 权限档位的 MCP API Key。现有 WorkBuddy 预约助手 Key 默认不包含收费、病历和回访日报权限。

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

### 写入医生排班

用户：把张医生 7 月 4 日和 7 月 5 日都排早班

Agent 步骤：

1. 调用 `list_doctors()` 找到张医生的 `doctor_id`
2. 调用 `list_schedule_shifts()` 找到“早班”的 `shift_id`
3. 向用户复述：张医生，2026-07-04、2026-07-05，设置为早班
4. 用户确认后调用 `batch_set_doctor_schedule(doctor_ids=[...], schedule_dates=["2026-07-04", "2026-07-05"], shift_id=..., user_confirmed=true)`
5. 按返回的 `summary`、`affected_count`、`doctor_names` 和 `dates` 回复

### 清空医生排班

用户：清掉张医生 7 月 4 日的排班

Agent 步骤：

1. 调用 `list_doctors()` 找到张医生的 `doctor_id`
2. 向用户复述：将清空张医生 2026-07-04 的排班
3. 用户确认后调用 `batch_set_doctor_schedule(doctor_ids=[...], schedule_dates=["2026-07-04"], shift_id=null, user_confirmed=true)`
4. 按返回的 `summary` 和 `affected_count` 回复

### 快速建档

用户：帮我给王小新建个档，手机号 13800138001，28 岁

Agent 步骤：

1. 先调用 `quick_create_patient(..., user_confirmed=false)` 检查手机号是否已有患者
2. 若返回 `created=false`，直接使用既有患者
3. 若需要新建档，向用户复述姓名、手机号、年龄、初诊日期
4. 用户确认后调用 `quick_create_patient(..., user_confirmed=true)`

### 创建预约

用户：给李测试约张医生 7 月 3 日 9:30 洁治 30 分钟

Agent 步骤：

1. 调用 `search_patients(keyword="李测试")` 确认 `patient_id`
2. 调用 `list_doctors()` 确认 `doctor_id`
3. 调用 `list_appointment_projects(keyword="洁治")` 确认 `project_id`
4. 调用 `find_available_slots(doctor_id=..., schedule_date="2026-07-03")` 确认目标时间空闲
5. 向用户复述预约信息
6. 用户确认后调用 `create_appointment(..., user_confirmed=true)`

### 运营日报

用户：帮我准备今天晨会的诊所运营情况

Agent 步骤：

1. 调用 `get_daily_clinic_operations(report_date="2026-07-02")` 获取总览数据
2. 如需预约明细，调用 `list_daily_appointments(scheduled_date="2026-07-02")`
3. 如需收费明细，调用 `list_daily_billing_records(report_date="2026-07-02", record_type="paid_records")`
4. 如需待收费，调用 `list_daily_billing_records(report_date="2026-07-02", record_type="pending_charges")`
5. 如需病历完成情况，调用 `get_daily_medical_record_report(report_date="2026-07-02")`
6. 如需回访安排，调用 `list_daily_followup_items(report_date="2026-07-02", focus="due_today")`
7. 根据工具返回的结构化数据生成日报正文，不要编造未返回的数据

## 认证方式

MCP Server 支持以下任一方式携带 API Key（命中其一即可）：

1. Query 参数：`?api_key=edmcp_xxx`
2. Header：`Authorization: Bearer edmcp_xxx`
3. Header：`X-EasyDental-Api-Key: edmcp_xxx`

内网 WorkBuddy 场景推荐使用 Query 参数，配置最简单。
