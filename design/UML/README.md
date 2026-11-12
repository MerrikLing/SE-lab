# lab2 UML 图清单

- 设计依据：`../顶层设计.md`
- 源文件：每个图同名 `.puml`；导出图：同名 `.png`
- 渲染：PlantUML 1.2023.13，Smetana 布局（本机无 Graphviz）；中文字体 Microsoft YaHei；导出 200 DPI
- 重新渲染：在本目录准备 `plantuml.jar` 后执行
  `java -Dfile.encoding=UTF-8 -jar plantuml.jar -tpng -charset UTF-8 -Sdpi=200 *.puml`
- 仓库未提交 `plantuml.jar`；可用 VS Code PlantUML 插件，或自行下载官方 jar

## 一、图清单与 lab2 分类

| # | 文件 | UML 类别 | 画什么 | 对应设计章节 |
|---|---|---|---|---|
| 1 | `UML类图-核心领域.puml` | 结构性（类图，必画） | User / Event / EventSession / TicketType / Registration / CheckIn 及关系 | §3、§4 |
| 2 | `UML类图-支撑领域.puml` | 结构性（类图） | Evaluation / Report / Favorite / NotificationMessage / Outbox / ExportRequest / AuditEntry | §4.5 |
| 3a | `UML类图-规则对象-状态与编辑.puml` | 结构性（类图） | 两套状态机、TransitionOverride、EditReviewPolicy、EventSnapshot/EventPatch | §4.2–4.4 |
| 3b | `UML类图-规则对象-权限与口径.puml` | 结构性（类图） | AccessContext/AccessPolicy、密码/登录节流、参与率/补签/代报/排序/脱敏 | §4.2–4.4 |
| 4 | `UML组件图.puml` | 结构性（组件图） | 8 个业务模块 + shared、依赖 DAG、外部适配器 | §2 |
| 5-1 | `UML用例图-参与者与组织者.puml` | 行为性（用例图） | 参与者 + 组织者用例（报告排版用） | §6、§11 |
| 5-2 | `UML用例图-检票员管理员与系统.puml` | 行为性（用例图） | 检票员 + 管理员 + 系统定时用例（报告排版用） | §6、§11 |
| 5-3 | `UML用例图.puml` | 行为性（用例图） | 四类角色 + 系统的全量总览（较高，屏幕查看用） | §6、§11 |
| 6 | `UML状态图-活动.puml` | 行为性（状态图） | 活动 6 状态与转换 | §8.1 |
| 7 | `UML状态图-报名.puml` | 行为性（状态图） | 报名 6 状态、候补终止、补签例外 | §8.2 |
| 8 | `UML活动图-报名流程.puml` | 行为性（活动图） | 报名校验、幂等、占座/候补、通知 | §7.1 |
| 9 | `UML序列图-报名与抢名额.puml` | 交互性（序列图） | 报名成功 + 并发抢最后 1 个名额 + 候补 | §7.1–7.2 |
| 10 | `UML序列图-取消活动.puml` | 交互性（序列图） | 取消活动、批量失效、退款、通知 | §7.5 |
| 11 | `UML序列图-扫码签到.puml` | 交互性（序列图） | 验签、分配校验、签到、重复扫码 | §7.4 |

lab2 最低要求：类图必画 + 三类各至少一种 + 总数 ≥4。本目录有 4 张类图、1 张组件图、6 张行为图（用例 3 + 状态 2 + 活动 1）、3 张交互图，满足并超过要求。

## 二、每张图的说明（可直接改写进报告）

### 1. 核心领域类图
划分三个聚合：`Event` 管活动生命周期与审核；`EventSession` 管场次容量与名额账本；`Registration` 管报名状态与签到。`TicketType` 是 `EventSession` 的聚合内实体，`CheckIn` 是 `Registration` 的聚合内实体。这样拆分的理由是并发热点不同：抢票只锁场次一行，审核和编辑不会与抢票争用同一个聚合。

### 2. 支撑领域类图
评价、举报、收藏、通知、导出、审计是低耦合的支撑能力。它们只按 ID 引用 `User` / `Event`，不反向修改核心聚合。评价以「场次 + 参与者」唯一且仅已签到可评；通知以 `dedupe_key` 去重；审计只追加。

### 3a. 规则对象：状态机与编辑重审
两套状态机（活动、报名）是状态转换的唯一合法入口；`TransitionOverride.MANUAL_CHECK_IN` 只允许补签用例传入。`EditReviewPolicy` 用 `EventSnapshot` 前后对比判断是否重审：票价/场次/容量/时间/地点属于结构性变更，标题/简介/封面属于文案变更。

### 3b. 规则对象：权限、统计口径与校验
`AccessContext` 只放扁平事实（owner / blacklisted / assigned / checked_in / visible），由用例查好后传入；`AccessPolicy` 是纯函数规则表，不查库。`ParticipationRatePolicy` 的分母是 `REGISTERED + CHECKED_IN + ABSENT`；`CheckInWindowPolicy` 固定 2 小时；`BeneficiaryPolicy` 上限 3 人；`SortPlan` 与 `ContactMasker` 负责排序白名单与手机号脱敏。

### 4. 组件图
8 个业务模块 + `shared`，箭头表示编译期依赖。`identity` 是最底层业务模块；`catalog` 依赖 `identity`；`registration` 依赖 `catalog`；`attendance` / `social` / `reporting` 依次向上；`notification`、`audit` 是叶子。`catalog` 取消活动需要批量失效报名，通过 `shared.RegistrationInvalidationPort` 倒置依赖，避免 `catalog ↔ registration` 环。为了可读性，图中省略所有模块到 `shared` 的边、外部入口（Web/UI、调度器）与 SQLite 适配器；完整 DAG 与外部依赖见设计文档 §2。

### 5. 用例图
四类角色：参与者、组织者、检票员、平台管理员；另加「系统定时任务」。「我的数据」类用例一律由 `ActorContext` 推导用户，接口不接收裸 `user_id`；组织者只能操作本人活动；检票员只能操作被分配场次；管理员不能代替组织者编辑活动。

### 6. 活动状态图
`DRAFT → PENDING_REVIEW → PUBLISHED → ENDED`；审核不通过进入 `REVIEW_REJECTED`，可重新提交或取消；已发布活动因票价/场次/容量/时间/地点等实质性修改回到 `PENDING_REVIEW`。`visibility`（LISTED/DELISTED）与 `status` 正交：下架只改可见性并写审计，不改变状态。

### 7. 报名状态图
六状态：候补中、已报名、已签到、已缺席、已取消、已失效。终态为后四个；唯一受控例外是 `ABSENT → CHECKED_IN`，仅限场次结束后 2 小时内人工补签，且必须记录操作人、原因并写审计。候补在活动取消时置 `INVALIDATED`，在场次结束时置 `CANCELLED(SESSION_ENDED)`。

### 8. 报名活动图
展示报名主流程：校验登录/可见/截止/黑名单 → 幂等键 → `SeatInventoryPort.claim` → 有名额创建 `REGISTERED` 并生成二维码，满员创建 `WAITLISTED` 并分配候补号 → 发件箱 + 审计 → 提交 → 通知。并发正确性由单条条件更新保证，不依赖应用层内存判断。

### 9. 报名与抢名额序列图
参与者 A 抢到最后 1 个名额，`SeatInventoryPort.claim` 的条件更新 `rowcount=1`，返回 `seat_no`；参与者 B 同时提交，`rowcount=0`，拿到 `waitlist_no` 进入候补。10 个并发下只有 1 条 `REGISTERED`；唯一索引 `(session_id, participant_id)` 防止重复有效报名。通知由发件箱异步投递，失败不回溯业务。

### 10. 取消活动序列图
组织者调用 `cancel_event`；`Event.assert_cancellable` 校验未开始；同一 UoW 内调用 `RegistrationInvalidationPort.invalidate_for_event`，逐条把有效报名置为 `INVALIDATED`，已支付记录调用模拟 `PaymentGateway.refund` 并置 `REFUNDED`；场次计数归零；发件箱生成 `EventCancelled`；提交后逐用户发站内消息。取消不因报名量被拒绝；原型单事务处理全部，超过 5000 条只告警。

### 11. 扫码签到序列图
验签在查库之前完成（HMAC 常数时间比较），伪造/篡改直接拒绝；校验二维码场次与扫码场次一致、检票员已分配；`REGISTERED → CHECKED_IN`，`RegistrationRepository.save` 级联保存 `CheckIn`；重复扫码返回「已签到」且不新增记录；`check_in` 表 `UNIQUE(registration_id)` 是并发双扫的最后防线。

## 三、图外的重要约束（报告文字需补充）

1. **名额唯一账本**：`EventSession.booked_count`、`next_waitlist_no`、`TicketType.sold_count` 只能由 `SeatInventoryPort` 的原子操作写；`EventSessionRepository.save_metadata` 禁写这些字段，容量修改走 `resize` 条件更新。
2. **一报名一签到**：`CheckIn` 是 `Registration` 聚合内实体，由 `RegistrationRepository.save` 级联持久化，`UNIQUE(registration_id)` 兜底。
3. **权限上下文**：`AccessContext` 只放扁平事实；`is_blacklisted`、`inspector_assigned`、`has_checked_in` 由用例查好后填入，规则不查库。
4. **通知映射**：`NotificationService.handle_event` 按 `NotificationType` 映射四类消息 + 开票提醒；`dedupe_key` 唯一，失败重试不重复。
5. **审计**：成功同事务、失败先回滚再独立事务写；主表只追加，365 天后归档而非删除。
6. **支付**：只有 `PAID / REFUNDED` 两个值；候补也置 `PAID`（模拟支付），取消候补/活动取消置 `REFUNDED`；参与者取消已确认报名不退款。

## 四、与设计文档的一致性检查

- **类图**：三聚合、`TicketType` 属场次、`CheckIn` 属报名、`Event` 与 `EventSession` 拆聚合，与 R5 §3/§4 一致；`Event.start_at/end_at/min_price` 标注为派生可空；`Registration.payment_status` 只有 `PAID/REFUNDED`；`AuditEntry.actor_id/actor_account/actor_role` 可空（至少一个非空）。
- **状态图**：活动六状态与审核回退、报名六状态、`WAITLISTED → CANCELLED(SESSION_ENDED)`、`WAITLISTED → INVALIDATED(EVENT_CANCELLED)`、`ABSENT → CHECKED_IN` 补签例外，与 R5 §8 一致。
- **组件图**：M1–M7 的业务依赖与 R5 §2 DAG 一致；省略“所有模块到 M0”的边、外部入口与 SQLite 适配器，原因写在图例；`M2 ..> M0 : RegistrationInvalidationPort` 对应依赖倒置。
- **活动图**：顺序为“解析 ActorContext → 幂等回放 → 权限/截止/黑名单 → claim”，与 R5 §7.1 的“幂等回放先于权限检查”一致。
- **序列图**：报名序列包含 `IdempotencyStore.get`，取消活动序列包含 `RegistrationInvalidationPort`/退款/`reset`，签到序列包含“验签在查库前/重复扫码幂等”，与 R5 §7 一致。
- **规则对象**：`AccessContext` 只含扁平事实；`EditReviewPolicy` 的 STRUCTURAL/COSMETIC 集合、参与率分母、补签 2 小时、代报 3 人，与 R5 §4 一致。
