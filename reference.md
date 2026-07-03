# MCP 工具参考

## 可用工具

| 工具名 | 说明 |
| --- | --- |
| `health_check` | 检查 MCP Server 是否正常 |
| `search_patients` | 按关键词搜索患者（手机号脱敏） |
| `get_patient_detail` | 按患者 ID 查询完整业务详情，手机号、身份证号和联系人电话脱敏 |
| `list_patients_for_analysis` | 分页读取分析用患者数据，每页最多 100 条 |
| `list_patient_appointments` | 查询患者预约列表 |
| `list_appointments_for_analysis` | 分页读取分析用预约数据，每页最多 100 条 |
| `get_current_clinic_profile` | 查询当前 API Key 绑定诊所 |
| `list_appointment_projects` | 查询可预约诊疗项目 |
| `quick_create_patient` | 按手机号查重并在确认后快速建档 |
| `create_appointment` | 在确认后创建普通预约 |
| `update_appointment` | 在确认后改约或更新预约摘要字段 |
| `cancel_appointment` | 在确认后取消预约并保留记录 |
| `check_in_appointment` | 在确认后将预约标记为已到店 |
| `mark_appointment_missed` | 在确认后将预约标记为失约 |
| `list_doctor_appointments` | 查询某医生日期范围内的预约 |
| `list_waiting_patients` | 查询某日候诊队列 |
| `list_doctors` | 列出在职医生 |
| `list_schedule_shifts` | 列出启用班次，供写入排班前选择班次 ID |
| `list_doctor_schedule` | 查询医生某天排班与占用 |
| `batch_set_doctor_schedule` | 在确认后批量设置或清空医生排班 |
| `find_available_slots` | 查询医生某天空闲时段 |
| `list_patient_medical_records` | 查询患者历史病历摘要 |
| `get_medical_record` | 读取单份病历详情 |
| `save_medical_record_draft` | 在确认后新建或更新病历草稿 |
| `list_followup_tasks` | 查询回访任务 |
| `update_followup_task` | 在确认后更新回访任务 |
| `record_followup_result` | 在确认后记录回访沟通结果 |
| `get_daily_clinic_operations` | 查询指定日期诊所运营总览 |
| `list_daily_appointments` | 分页查询某日预约明细 |
| `list_daily_billing_records` | 分页查询某日收费、待收费或退款记录 |
| `list_billing_records_for_analysis` | 分页读取分析用账单、收费、退款或待收费数据；`record_type` 可空，默认账单；每页最多 100 条 |
| `get_daily_medical_record_report` | 查询某日病历状态统计和待补病历候选 |
| `list_daily_followup_items` | 分页查询某日回访待办、已回访、逾期、需医生处理或明日待办 |
| `get_date_range_operations_report` | 查询日期范围经营汇总 |
| `get_medical_record_workload_report` | 查询病历工作量和待补病历 |
| `get_doctor_performance_report` | 查询医生固定口径工作量试算 |

> 完整收费、退款明细和运营日报收费板块需要使用 `agent_daily_reporter` 权限档位的 MCP API Key。MCP 不执行收费、退款、库存管理、收费单打印或患者来源维护。

## 示例对话

### 查患者

用户：帮我找一下叫张三的患者

Agent 步骤：

1. 调用 `search_patients(keyword="张三")`
2. 若只有 1 条，可直接展示摘要
3. 若用户需要业务详情、过敏史、标签或分组，再调用 `get_patient_detail(patient_id=...)`
4. 注意患者详情中的手机号、身份证号和备用联系人电话仍为脱敏值

### 患者分析

用户：分析一下本月建档患者的来源和年龄结构

Agent 步骤：

1. 明确分析时间口径，例如 `date_field="created_at"`、`date_start="2026-07-01"`、`date_end="2026-07-31"`
2. 调用 `list_patients_for_analysis(page_size=100, ...)`；如果用户没有给筛选条件，直接空参数查询，不要猜日期或来源
3. 先读取返回的 `field_schema`、`analysis_notes`、`privacy_notes` 和 `pagination_notes`，按中文标签理解字段
4. 如果返回 `has_next_page=true`，继续用 `next_page` 调用，直到 `has_next_page=false`
5. 只基于完整匹配集做统计；如果无法拉全，必须在结论中说明“数据未完整获取”
6. 不使用 `search_patients` 的搜索结果样本做患者结构分析

### 查预约

用户：看一下 patient-001 最近有哪些预约

Agent 步骤：

1. 调用 `list_patient_appointments(patient_id="patient-001")`
2. 按返回的 `summary` 和 `items` 组织回复

### 预约分析

用户：分析一下本月预约来源渠道和失约情况

Agent 步骤：

1. 明确分析时间口径，例如 `date_field="scheduled_start"`、`date_start="2026-07-01"`、`date_end="2026-07-31"`
2. 调用 `list_appointments_for_analysis(page_size=100, ...)`；如果用户没有给筛选条件，直接空参数查询，不要猜日期、状态或医生
3. 先读取返回的 `field_schema`、`analysis_notes`、`privacy_notes` 和 `pagination_notes`，按中文标签理解字段
4. 如果返回 `has_next_page=true`，继续用 `next_page` 调用，直到 `has_next_page=false`
5. 基于完整预约匹配集统计状态、渠道、医生、项目；不要只取 `list_daily_appointments` 的单日明细做跨期结论

### 收款分析

用户：分析一下本月实收、退款和待收费

Agent 步骤：

1. 分别按需要调用 `list_billing_records_for_analysis(record_type="orders" | "payments" | "refunds" | "pending_charges", page_size=100, ...)`；如果用户只说“查收费/账单数据”且没有筛选条件，可先空参数调用，默认返回账单全量第一页
2. 收费单通常使用 `date_field="paid_at"`；退款使用 `date_field="refunded_at"`；账单使用 `date_field="billing_day"`
3. 每个 `record_type` 先读取返回的 `field_schema`、`analysis_notes`、`privacy_notes` 和 `pagination_notes`，按中文标签理解字段
4. 每个 `record_type` 都按 `has_next_page` / `next_page` 拉取完整匹配集后再计算
5. 说明收费签名和收款凭证号不作为分析字段返回；待收费是派生记录，需区分 `unpaid_order` 和 `unpriced_appointment`

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

### 改约

用户：把李测试明天上午的预约改到 10 点

Agent 步骤：

1. 调用 `search_patients(keyword="李测试")` 确认患者
2. 调用 `list_patient_appointments(patient_id=...)` 找到目标 `appointment_id`
3. 如需指定医生，调用 `find_available_slots(doctor_id=..., schedule_date=...)` 确认 10 点空闲
4. 向用户复述原预约和新时间
5. 用户确认后调用 `update_appointment(appointment_id=..., scheduled_date=..., start_time="10:00", user_confirmed=true)`

### 预约签到或失约

用户：李测试到了，帮我签到

Agent 步骤：

1. 调用 `list_patient_appointments(patient_id=...)` 或 `list_doctor_appointments(...)` 找到目标预约
2. 向用户复述患者和预约时间
3. 用户确认后调用 `check_in_appointment(appointment_id=..., user_confirmed=true)`

用户：张医生 9 点那个患者没来

Agent 步骤：

1. 调用 `list_doctor_appointments(doctor_id=..., start_date=..., end_date=...)` 找到目标预约
2. 向用户复述患者和预约时间
3. 用户确认后调用 `mark_appointment_missed(appointment_id=..., user_confirmed=true)`

### 病历草稿

用户：给李测试保存一份草稿，主诉是右下后牙疼两天，诊断疑似龋齿

Agent 步骤：

1. 调用 `search_patients(keyword="李测试")` 确认患者
2. 如有关联预约，调用 `list_patient_appointments(patient_id=...)` 确认预约
3. 向用户复述患者、医生、就诊时间、主诉和诊断
4. 用户确认后调用 `save_medical_record_draft(..., user_confirmed=true)`
5. 不调用提交、作废或删除病历能力

### 回访

用户：看一下今天待回访

Agent 步骤：

1. 调用 `list_followup_tasks(due_start="2026-07-02", due_end="2026-07-02", status_filter="pending")`
2. 按返回的 `summary` 和 `items` 回复

用户：这个回访已接通，患者恢复正常，结束任务

Agent 步骤：

1. 确认目标 `followup_task_id`
2. 向用户复述联系方式、联系结果、沟通内容和下一步动作
3. 用户确认后调用 `record_followup_result(contact_result="connected", next_action="complete", user_confirmed=true)`

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

### 多日汇总

用户：看一下 7 月上半月医生工作量和病历待补

Agent 步骤：

1. 调用 `get_doctor_performance_report(start_date="2026-07-01", end_date="2026-07-15")`
2. 调用 `get_medical_record_workload_report(start_date="2026-07-01", end_date="2026-07-15")`
3. 只按工具返回的固定口径解释，不把试算结果直接当成最终绩效结论

## 认证方式

MCP Server 支持以下任一方式携带 API Key（命中其一即可）：

1. Query 参数：`?api_key=edmcp_xxx`
2. Header：`Authorization: Bearer edmcp_xxx`
3. Header：`X-EasyDental-Api-Key: edmcp_xxx`

内网 WorkBuddy 场景推荐使用 Query 参数，配置最简单。
