# 曾文 - AI Agent 方向面试高频问题准备

> 生成时间：2026-07-27  
> 岗位：AI Agent 后端工程师  
> 核心项目：PaiCLI-Python / PaiAgent / PaiSmart / 大宅门推客系统

---

## 一、PaiAgent 项目

### Q1：PaiAgent 的双引擎架构是什么？DAG vs LangGraph 怎么选？

**一句话定位：**
PaiAgent 是"一个调度层，两套执行策略"，EngineSelector 根据 `workflow.engineType` 路由，默认 DAG，LangGraph 处理复杂状态循环场景。

**精简版（60 秒）：**
> "PaiAgent 双引擎通过 EngineSelector 路由，默认 DAG。DAG 拓扑排序、无环、线性执行，简单好调试；LangGraph 用状态机 + 条件边处理循环和复杂决策。两个引擎共享 NodeExecutor 接口，LangGraph 通过 NodeAdapter 适配执行，所以加新节点类型两个引擎自动能用。"

**完整版（90-120 秒）：**
> "PaiAgent 双引擎设计是'一个调度层，两套执行策略'。EngineSelector 根据 workflow.engineType 选择引擎，默认 DAG，LangGraph 处理复杂场景。
>
> DAG 引擎适合固定流水线：拓扑排序在运行前就确定执行顺序，Kahn 算法 + DFS 环检测保证无环，天然适合 Input → LLM → Output 这种线性工作流。它的优势是简单、可预测、好调试，而且支持断点续执行。
>
> LangGraph 引擎适合需要循环和状态的场景。比如一个工作流：先让 LLM 判断是否需要查数据库，查完再判断是否需要进一步拆分任务，拆完再循环执行。这种'条件边 + 循环' DAG 做不了，因为 DAG 是无环的，执行顺序编译期就写死了。
>
> 关键是我们没有重写两套节点执行器。NodeExecutor 接口是共享的，LangGraph 通过 NodeAdapter 把现有节点适配成 AsyncNodeAction，所以新增节点类型时两个引擎都能用。"

**面试金句：**
> "PaiAgent 的双引擎不是两套实现，而是共享同一套节点执行器，只有调度层不同。"

---

### Q2：讲一个你解决的复杂 Bug（VolcengineAgentPlanClient 空错误流）

**STAR 结构：**

- **S（Situation）**：在本地跑 PaiAgent，验证 ReAct Agent 工作流。输入是"记住我叫张三，问我今天星期几"，运行后发现 Agent 反复尝试调用 `memory_write`，每次都报模糊的 `invoke` 错误，DebugDrawer 看不到真实原因。
- **T（Task）**：定位为什么错误信息被吞掉，找到真正的 embedding 报错。
- **A（Action）**：定位到 `VolcengineAgentPlanClient.java`。原代码：
  ```java
  byte[] bytes = (status >= 200 && status < 300 ? conn.getInputStream() : conn.getErrorStream()).readAllBytes();
  ```
  当 HTTP 失败且 `getErrorStream()` 返回 null 时，`.readAllBytes()` 触发 NPE，把真实错误盖住了。
  修复：
  ```java
  InputStream stream = (status >= 200 && status < 300) ? conn.getInputStream() : conn.getErrorStream();
  byte[] bytes = stream != null ? stream.readAllBytes() : new byte[0];
  ```
- **R（Result）**：修完后再次运行，直接看到了 `memory_write` 的真实错误——embedding 接口配置缺失。

**面试金句：**
> "这个改动虽然只改了两三行，但它把所有后续 Agent Plan 工具调用的排查成本从盲猜变成看真实的 HTTP 状态码和错误响应，是一个小但关键的防御性改进。"

**可能追问：**
- 如果 errorStream 为空，你还能从哪获取错误信息？答：HTTP status code、response message、headers。
- 这种 bug 怎么避免？答：HTTP 客户端对输入流判空是基本防御，代码审查时关注 `.getErrorStream()`、`getInputStream()` 等可空 API。

---

### Q7：PaiAgent 的工具注册与发现机制是怎么设计的？

**一句话定位：**
三层架构。启动时 AgentToolRegistry 全量注册（内置 + MCP），运行时按工具名查实例拼进 LLM tools 参数，前端问题在硬编码没动态拉取。

**口述答案（90 秒）：**
> "PaiAgent 工具分三层。
>
> 第一层注册：启动时 AgentToolRegistry 扫描 Component 注册内置工具（如 memory_write），同时从 mcp_tool_config 表加载 MCP 工具，全量存在 registry 里。
>
> 第二层调用：ReActAgentNodeExecutor 拿到节点配置的工具名列表，去 registry 查出 AgentTool 实例，把它的 name / description / inputSchema 拼进 LLM 调用的 tools 参数。LLM 返回 tool_calls 时，executor 按名从已查好的实例池里取出来执行。
>
> 第三层展示：问题在前端 EditorPage.tsx 硬编码了 memory_write 等几个工具，没调用后端接口动态拉取全部工具。所以用户配置了 MCP Server，工作流里也选不到。"

**面试金句：**
> "PaiAgent 后端已经支持 MCP 扩展工具，限制主要在前端的展示层。"

**可能追问：**
- MCP 工具怎么注册进来的？答：用户在前端配置 MCP Server 信息，存到 mcp_tool_config 表，后端重启或手动刷新时拉取注册。
- 怎么改进前端展示层？答：后端新增 /api/agent-tools 接口返回全部工具 metadata，前端工具面板改成动态拉取，MCP 配置后自动刷新。
- 权限怎么控制？答：加一层权限过滤，不同工作流/用户只能看到授权工具。

---

### Q8：PaiAgent 的 ReAct Agent 运行时流程是怎样的？

**一句话定位：**
for 循环 + LLM 调用 + JSON 决策解析 + finish_reason 判断，默认最多 5 步，上限 20 步。

**口述答案（60 秒）：**
> "PaiAgent 的 ReAct Agent 在 ReActAgentNodeExecutor 里是一个 for 循环，默认最多 5 步，上限 20 步。
>
> 每步：executor 把从 AgentToolRegistry 查出来的工具 schema 拼进 LLM 调用的 tools 参数。LLM 返回一个 JSON 决策，executor 用 JSONObject.parseObject() 解析，提取 action（工具名）和 action_input（参数）。
>
> 然后判断：如果 LLM 的 finish_reason == 'stop'，说明 LLM 认为不需要调工具了，直接返回最终答案；如果 finish_reason == 'tool_calls'，说明 LLM 要调工具，就去 registry 查对应工具实例来执行，结果再塞回 messages，继续下一轮循环。
>
> 步数到了上限还没结束，就强制终止，返回当前结果。"

**面试金句：**
> "ReAct Agent 的'智能'来自循环 + 工具调用失败反馈。Agent 它不是一次想对，而是能根据反馈调整，直到找到正确答案或达到上限。"

**可能追问：**
- 怎么防止死循环？答：max_steps 硬上限（20 步）。
- 如果 LLM 返回的 JSON 格式不对怎么办？答：try-catch 解析异常，报错并终止当前步，继续下一轮。
- token 浪费怎么控制？答：每步 messages 都会增长，可以通过 max_tokens 限制单次输出长度。

---

## 二、公司项目：大宅门推客系统

### Q3：AI 数据图表直查系统是怎么设计的？为什么绕过 LLM？

**一句话定位：**
规则前置，LLM 兜底。高频数据查询走正则意图匹配 + 预置 SQL，不调用 LLM；非数据意图走 LLM SSE 流式回复。

**口述答案（90-120 秒）：**
> "我在公司项目里做了一个 Data Chart Router，思路是'规则前置，LLM 兜底'。
>
> 第一层是规则命中：用户问'本月销售额''今日新增线索'这类高频中文查询，我用正则做意图匹配，命中后直接查 PostgreSQL 生成 SVG/PNG 返回，完全不调用 LLM。这样响应快、无幻觉。
>
> 第二层是规则未命中但听起来像查数据：给用户一个友好回复'这个查询暂时不支持，已记录'，后台异步补规则。
>
> 第三层是非数据类闲聊：比如'早上好'，走 OpenClaw Gateway 的 LLM SSE 流式回复。
>
> Trade-off 是开发侧要维护规则表和 SQL，但换来了低延迟和高确定性。另外我们对不同指标做了分级缓存，图表标注上次更新时间，让用户自己判断数据是否新鲜。"

**面试金句：**
> "生产级 AI 系统的核心不是让 LLM 做所有事，而是让 LLM 只做它该做的事。"

**可能追问：**
- 规则怎么维护？答：运营/产品提供高频查询模板，后端维护正则 + SQL 映射。
- LLM 写 SQL 有什么问题？答：幻觉、权限风险、复杂 SQL 不稳定、延迟高。
- 如果用户问"上个月和这个月销售额对比"这种复合查询怎么办？答：拆成多个单表查询或预置复合 SQL，不在生产环境让 LLM 动态拼 SQL。

---

### Q4：意图匹配具体怎么做？正则之间冲突怎么办？

**一句话定位：**
40+ 条正则按“具体意图优先于笼统意图”的顺序匹配，配合 `normalize` 和负向前瞻，避免“本月业绩冠军”被 `monthly_kpi` 抢走。

**口述答案（90 秒）：**
> “意图匹配分两层。第一层是 `PATTERNS` 有序正则：覆盖今日/本周/本月新增线索、签约、进店、推荐官排名、个人业绩、本周/本月签约冠军与最差员工等高频问法。每条规则只负责一个明确意图，匹配上后直接调用对应的 `query_xxx` 函数。
>
> 冲突靠顺序解决。比如 `personal_performance` 要排在 `monthly_kpi` 之前，否则‘李海芳本月业绩’会先命中整体看板；`monthly_signed_champion` 也要排在 `monthly_kpi` 之前，否则‘本月业绩冠军’会返回整月 KPI。
>
> 第二层是 `is_data_related`：没命中任何规则但包含‘线索/业绩/排名/多少’等数据词时，直接返回‘暂不支持，已记录’，不再丢给 LLM 耗时间。”

**面试金句：**
> “正则顺序就是业务优先级，越具体的问题越要靠前。”

**可能追问：**
- 怎么加新意图？答：加一条 `PATTERNS` 正则、一个 `query_xxx` 函数、再注册到 `QUERY_DISPATCH`，必要时调整顺序。
- 怎么防止误匹配？答：用负向前瞻避免时间词被当成人名；用测试集回归常见问法。
- 用户问法千变万化怎么办？答：先把高频问法固化成规则，长尾问题走记录→补规则，而不是让 LLM 现场写 SQL。

---

### Q5：怎么保证 SQL 只读、安全？

**一句话定位：**
三道防线：预置 SQL 本身只读、运行期关键字白名单校验、数据库账号最小权限。

**口述答案（90 秒）：**
> “所有命中规则走的都是代码里写死的 SELECT，不会拼接写操作。
>
> 如果将来走 LLM 生成 SQL，也有 `validate_sql` 做运行期校验：必须以 `SELECT` 开头；禁止 INSERT/UPDATE/DELETE/DROP/CREATE/ALTER/TRUNCATE/GRANT/EXECUTE/CALL/COPY/LOAD/VACUUM；禁止分号和注释符。最后 `run_sql` 通过 SSH BatchMode 连到 dzm 服务器，把 SQL base64 传给 psql 只读执行。
>
> 另外生产账号本身只做查询权限，修改资金类数据必须走 `docs/maintenance/` 留痕，这是项目宪章里的强制规范。”

**面试金句：**
> “数据查询的信任链：代码层校验 + 协议层只读 + 账号层最小权限。”

**可能追问：**
- 如果 LLM 绕过校验怎么办？答：目前数据类未命中直接返回不支持，不调用 LLM；若以后开放 LLM SQL，也只在白名单校验通过后才执行。
- 注入风险？答：规则查询是预置模板；参数化查询优先，字符串拼接的字段会做白名单过滤。

---

### Q6：缓存和性能是怎么设计的？本地硬件不够怎么办？

**一句话定位：**
分级 TTL + 参数化 cache key + 慢查询先给即时反馈，用规则和缓存抵消本地 LLM 算力不足。

**口述答案（90 秒）：**
> “我们为不同意图设置了分级缓存 TTL：今日实时指标 5 分钟，周/月趋势 15 分钟，分布/状态类 30 分钟。`cache_key` 会把参数也算进去，比如人名、时间范围、指标，避免‘李海芳本月业绩’和‘李海芳上周业绩’互相覆盖。
>
> 用户说‘刷新/最新/重新查’时强制穿透缓存。个人业绩查询如果 SQL 较慢，桥接器会先回一条‘正在查询 xxx，请稍候…’，再后台跑 SQL，避免用户盯着‘正在思考中…’。
>
> 因为本地模型跑 SQL 生成很慢，我们的核心策略是‘规则前置，LLM 兜底’：高频数据查询完全不调用 LLM，只有闲聊才走 OpenClaw Gateway 的 SSE 流式回复。”

**面试金句：**
> “在有限硬件下，缓存和规则是延迟的敌人；LLM 只留给真正需要推理的对话。”

**可能追问：**
- 缓存怎么失效？答：TTL 到期自动失效，用户也可以主动说‘刷新最新数据’。
- 图片外网加载不了怎么办？答：桥接器通过 `send_image` 让 server.py 走企微临时素材上传，不再依赖内网 `192.168.0.253:18082` 的 URL。

---

### Q7：怎么支持“查具体的人”？模糊匹配怎么做？

**一句话定位：**
把人名、时间、指标三要素从问法里拆出来，先模糊匹配用户表，再按参数查业绩。

**口述答案（90 秒）：**
> “`extract_person_name` 会先去掉 `@HemesBot` 这种企微提及、前缀动词、时间词和后缀，得到干净的人名。`extract_time_label` 和 `extract_metric` 分别解析时间范围和指标，默认本月、综合业绩。
>
> `query_personal_performance` 里用 `real_name ILIKE '%name%' OR nickname ILIKE '%name%'` 做模糊匹配：
> - 没找到，返回‘未找到该员工，请检查输入’；
> - 找到多个，返回候选列表，让用户补全；
> - 找到唯一，再按时间范围查负责线索、签约、开工、定金、进店、跟进等。
>
> SQL 里同时查 `assigned_assistant_id` 和 `assigned_designer_id`，所以店小二和设计师都能查到。”

**面试金句：**
> “把自然语言先翻译成结构化参数（人名+时间+指标），再交给预置 SQL，而不是让 LLM 去理解。”

**可能追问：**
- 同名怎么办？答：返回候选列表，让用户补全姓名。
- “跟单量”是什么意思？答：在这个系统里把“跟单量/跟单”映射为“负责线索”，表示该员工名下跟进的线索数量。

---

### Q8：如果用户的数据问题没命中规则，系统会怎么处理？

**一句话定位：**
数据类未命中直接给友好提示并记录，不私下调用 LLM 跑查询，避免慢响应和幻觉。

**口述答案（60 秒）：**
> “当规则没命中但 `is_data_related` 判断是数据类问题时，桥接器会返回一段固定提示：‘该数据查询暂时不支持自动出图，已记录，后续会补充’，并列出当前支持的示例。这样用户不会空等，也知道能问什么。
>
> 非数据类闲聊才走 OpenClaw Gateway 的 SSE 流式回复。这个设计的关键是‘不为了回答而回答’——如果规则覆盖不了，宁可诚实告知，也不让本地小模型现场编 SQL。”

**面试金句：**
> “生产级系统的底线是：不能回答的问题，要明确拒绝，而不是给出一个看起来对但可能错的答案。”

**可能追问：**
- 未命中的问题怎么变成规则？答：收集日志里的高频未命中问法，按业务语义抽象成正则 + SQL，再注册到 `PATTERNS`。
- 会不会漏掉用户真正的长尾需求？答：会定期 review 未命中日志，对确实有价值的长尾查询补规则；完全个性化的一次性查询不纳入规则。

---

### Q9：大宅门推客系统整体架构是什么？你负责哪些模块？

**一句话定位：**
家装行业线索分销与运营平台，FastAPI v2 异步栈 + PostgreSQL + Redis，142+ API，多角色 + 状态机 + 佣金支付 + AI 数据直查。

**口述答案（90-120 秒）：**
> "大宅门推客系统是给家装公司用的线索分销和运营平台。业务上就是推荐官拉客户进来，线索流转过店小二跟进、设计师进店、定金、签约、开工、完工这条链，每个环节都有对应的佣金奖励。
>
> 技术栈是 Python 3.12 + FastAPI 0.115 + SQLAlchemy 2.0 async，数据库 PostgreSQL 16，缓存 Redis 7。系统从 v1 单体 Django 重构到 v2 异步 FastAPI，一共 43 个路由模块，142+ 接口，跑了 1400+ 次 git 提交。
>
> 我负责的模块主要有四个：
> 1. **权限与数据隔离**：把 v1 里散落在每个接口的硬编码角色判断收拢成 RequirePermission + RequireCertified + RequireIdentityVerified 三层守卫，再加 CompanyScope 公司级数据隔离，总公司自动聚合所有分公司数据。
> 2. **线索状态机重构**：把线索状态从单值改成了 17 位 JSONB 状态标记，奖励关联位天然幂等，主管恢复不会重复发奖。
> 3. **微信支付佣金对账**：设计多级佣金链路，提现 ≤400 元直接微信转账、>400 元走主管审核。最关键的是三次幂等保障：支付回调幂等、提现状态幂等、金额不一致告警，还有一个每 5 分钟循环的主动对账。
> 4. **v1 → v2 生产迁移**：我写了 17 步 Alembic 迁移脚本，完成全量生产数据迁移并做了充分验证。"

**面试金句：**
> "这个项目让我把 Python 异步工程 + 业务状态机 + 资金安全三个维度同时落地在一个真实系统中。"

**可能追问：**
- 重构前后有什么核心差异？答：v1 Django 同步，接口里权限散落、状态是单字段，耦合重；v2 FastAPI 异步，权限集中，状态位灵活扩展，各层通过依赖注入解耦。
- 技术选型为什么用 FastAPI 而不是 Django REST？答：FastAPI 的 async/await 对 PostgreSQL 高并发支持更好，Pydantic 的声明式验证比 Django 序列化器更严谨。

---

### Q10：线索的 17 位状态标记是怎么设计的？为什么不用传统状态机？

**一句话定位：**
用 JSONB 存 17 个布尔位，每个位代表一个可独立翻转的状态维度，解决多对多状态组合和奖励耦合问题。

**口述答案（90 秒）：**
> "传统做法是用一个字符串 status 表示线索当前状态，比如 'visited'。问题是什么？当一个线索同时满足'已验证'+'已分配'+'跟进中'+'已进店'时，一个字符串只能表达一个维度。而且奖励触发和状态耦合很深——如果主管把一条'已签约'的线索恢复成'跟进中'，再签一次约时，因为 status 字段值变了，很难判断奖励是否已经发过。
>
> 我的方案是把状态拆成 17 个独立的布尔位，存在 JSONB 里。核心位包括 verified、director_confirmed、assigned、following、visited、deposited、signed、started、completed，以及三个奖励关联位：reward_issued（推荐奖励已发）、reward_visit_issued（进店奖励已发）、reward_claimed（已领取）。
>
> 关键设计是：奖励位和业务位是独立的。signed 可以从 0 到 1 多次翻转，但 reward_issued 设置后就永远不变。所以主管恢复线索不会重复发奖。
>
> 对外展示的字符串状态通过 `get_current_status` 按优先级推导：escaped > invalid > completed > started > signed > deposited > visited > following_up > confirmed > pending_verify。这样数据库存的是多维位图，API 返回的是一个可读的状态字符串，两者解耦。"

**面试金句：**
> "当业务状态不是线性流转而是多维组合时，布尔位图比状态字符串更灵活——特别是状态变化会触发资金操作时。"

**可能追问：**
- 17 位会不会太复杂？答：核心业务位只有 10 个，其余是奖励和扩展位。实际使用时只需要关心和自己业务相关的几个位，复杂度可控。
- 怎么防止位冲突？答：`set_flag` 内部是幂等的——从 0 到 1 返回 True（触发奖励），从 1 到 1 不重复触发。
- 旧数据怎么迁移？答：`build_status_flags` 函数可以从旧 status 字符串推导出完整的 flags 字典，兼容迁移。

---

### Q11：权限三层守卫 + CompanyScope 是怎么实现的？为什么 v1 有这个需求？

**一句话定位：**
用 FastAPI 依赖注入实现 RBAC + 认证检查 + 实名校验三层链式守卫，CompanyScope 在 ORM 层做公司级数据过滤。

**口述答案（90 秒）：**
> "v1 的问题是每个接口自己 `if user.role == 'assistant' ... else ...` 硬编码权限，改了角色名就要改几十个接口。
>
> v2 我设计了集中式权限体系：
> - **第一层 RequirePermission**：声明式 RBAC，比如 `Depends(RequirePermission('assistant', 'store_manager'))`。内部维护一个 frozenset 的角色白名单，不在白名单直接 403。
> - **第二层 RequireCertified**：设计师资质，查 certificationStatus == 'approved'，没通过认证不能接单。
> - **第三层 RequireIdentityVerified**：实名校验，查询 IdentityVerification 表中有 FaceID 或手机号的 verified 记录。主要是提现前防身。
> - 另外还有 **RequireBalanceRole**，专门拦截设计师和店小二访问余额/提现接口。
>
> 三层是链式关系：先过角色白名单，再过资质，最后过实名。每个守卫都可以独立组合，比如 update_status 接口同时加了 RequireCertified 和 RequirePermission。
>
> CompanyScope 是在 ORM 层的公司数据隔离。每次查询前自动追加 `WHERE company_id IN (...)`：如果是分公司用户，只查本公司；如果是总公司用户，自动查所有分公司。这样总公司管理层看到的是全公司聚合数据。"

**面试金句：**
> "权限不是'散落在每个接口的 if-else'，而是'一列依赖注入链，谁加谁摘都一目了然'。"

**可能追问：**
- 如果用户属于多个公司怎么办？答：`get_current_user` 返回 user 的单个 company_id，CompanyScope 用 users.company_id 过滤。目前业务模型是单公司归属。
- 权限守卫的顺序有要求吗？答：FastAPI 依赖注入自然形成链式调用，通常先 RequirePermission（最快失败），再 RequireCertified，最后 RequireIdentityVerified（需要查库）。
- 怎么加一个新角色？答：在 RequirePermission 初始化时传新角色名，其他代码不用改。

---

### Q12：微信支付佣金和提现怎么做幂等？对账怎么设计？

**一句话定位：**
三重幂等保障 + 每 5 分钟主动对账轮询，先 commit 再调微信防止钱单两失。

**口述答案（90-120 秒）：**
> "提现链路分小额和大额两个分支：≤400 元直接微信转账，>400 元进 pending_audit 等主管审核。整个链路围绕资金安全做了几层保障：
>
> **并发控制**：提现时用 `select(Balance).with_for_update()` 行锁 + 排他检查，防止同一个人同时发起两笔提现。
>
> **顺序保障（先 commit 再调微信）**：这是我从踩坑中定下来的原则。如果把调微信放在 commit 之前，万一 commit 失败了但微信已经转账成功，钱出去了系统没记录。所以流程是先冻结余额、创建 Withdrawal 记录、commit 到数据库，再调用微信转账。微信失败了立即解冻退款。
>
> **幂等三重保障**：
> 1. 回调幂等：用微信通知的 notify_id 做唯一键去重，同一条回调只处理一次。
> 2. 状态幂等：处理提现回调时先 `select...with_for_update()` 锁住行，如果状态已经是 success/failed 就跳过。
> 3. 金额校验：回调里的金额和数据库记录不一致时拒绝状态变更，输出 critical 日志等人工介入。
>
> **主动对账**：回调不是 100% 可靠的。我用 APScheduler 启动了一个每 5 分钟的任务，扫描所有 status=processing 且有微信转账单号的提现记录，逐条调微信查询接口。微信返回 SUCCESS 就确认到账，FAIL 就退款。另外还有滞留告警：processing 超 30 分钟或 pending 超 10 分钟，出 error 日志通知人工排查。"

**面试金句：**
> "支付系统做好三层事：并发不超扣、回调不重复处理、余额不一致时宁可停等人工也不自动修。"

**可能追问：**
- 如果主动对账和回调同时到达怎么办？答：行锁 `with_for_update()` 保证串行，先到的先处理，后到的幂等跳过。
- 大额提现的主管审核怎么触发？答：目前是线下通知主管到后台审核。技术上是创建 pending_audit 状态记录，审核接口把状态改成 processing 后触发微信转账。
- 对账跑太久会影响性能吗？答：每轮只查 processing 状态的记录，量很小（通常只有几条），不会产生慢查询。
- 为什么不用消息队列？答：项目规模不需要，5 分钟定时任务够用。如果提现量大了，可以用 RabbitMQ 的延迟队列替代。


---

## 三、通用基础

### G4：MCP vs Function Calling 的区别？

**一句话定位：**
Function Calling 是 LLM 调用本地函数的能力；MCP 是让 LLM 连接外部工具/服务的标准协议。

**口述答案（90 秒）：**
> "Function Calling 是 LLM 本身的能力：模型看到函数定义后，输出调用参数，由应用层直接执行本地函数。它适合单应用、单进程场景，简单直接。
>
> MCP 是一个标准协议，定义了 LLM 客户端怎么和外部工具服务通信。它把工具从 LLM 应用里抽出来，放到独立的 MCP Server 里。比如一个文件系统 MCP Server、一个 PostgreSQL MCP Server，多个 Agent 都可以连。
>
> 核心区别三点：1. Function Calling 是调用能力，MCP 是连接协议；2. Function Calling 的函数在应用内部，MCP 的 Server 是独立进程/服务；3. MCP 的工具可以跨应用复用，Function Calling 的工具通常只服务当前应用。
>
> 代价是 MCP 需要单独部署 Server，Function Calling 更轻量。"

**面试金句：**
> "Function Calling 是 LLM 怎么调用函数；MCP 是怎么让 LLM 安全地调用外部系统的工具。"

**可能追问：**
- PaiCLI-Python 里怎么接入 MCP？答：支持 stdio 和 Streamable HTTP 两种传输方式。stdio 适合本地工具进程，HTTP 适合远程服务。
- MCP Server 的权限怎么控制？答：MCP 协议本身有权限申请机制，Server 声明工具，Client 使用前请求用户授权。
- 什么场景选 Function Calling，什么场景选 MCP？答：单应用内部工具用 Function Calling；需要跨应用共享、进程隔离、第三方生态的工具用 MCP。

---

### G5：RAG 全流程 + Chunk 策略

**一句话定位：**
RAG 是“检索外部知识 → 拼进上下文 → LLM 生成”的流水线，Chunk 策略按文本结构、成本和语义质量来选。

**口述答案（90-120 秒）：**
> "RAG 全称 Retrieval-Augmented Generation，解决 LLM 知识滞后和幻觉。
>
> 完整流程分六步：
> 1. **文档解析**：把 PDF、Word、网页等转成干净文本。
> 2. **Chunk 切分**：按策略切成语义完整的块。
> 3. **Embedding**：把 chunk 转成向量。
> 4. **向量库存储**：向量 + 原始文本 + 元数据一起存到向量数据库。
> 5. **检索**：用户问题也走 embedding，算相似度，召回 top-k，必要时重排序。
> 6. **生成**：把检索结果作为上下文拼进 prompt，LLM 基于证据回答。
>
> Chunk 策略常见四种：
> - **固定字符/token**：简单，但可能截断语义。
> - **递归切分**：先按段落、再按句子，保留自然边界。
> - **语义切分**：用 embedding 找语义转折处切，质量高、成本高。
> - **结构切分**：代码按函数/类，文档按 Markdown 标题。
>
> 选型原则：有结构用结构切分；连续长文用递归切分；对质量要求极高且算力够，用语义切分。
>
> PaiCLI-Python 里我做了一个轻量版 CodeIndex：按代码文件的每一非空行做 chunk，存到 SQLite，查询用 LIKE 关键词匹配。它不是向量 RAG，但代码检索对精确关键词更敏感，所以这样够用。"

**面试金句：**
> "RAG 不是只有向量检索，只要是'先检索外部知识、再让 LLM 生成'，就是 RAG。"

**可能追问：**
- chunk 大小一般多少？答：256-1024 token，看语义完整性。
- 检索不准怎么办？答：加元数据过滤、重排序、混合检索（向量 + 关键词）。
- PaiSmart 的 RAG 怎么做的？答：Spring Boot + ES + MinIO，文档解析后走 ES 检索，具体细节我主要熟悉设计层面。

---

### G6：PaiCLI-Python 的记忆系统和 Context 压缩

**一句话定位：**
三层记忆：项目记忆文件放静态约定，SQLite 长期记忆库存跨会话事实，Agent 内 history 维护短期对话；Context 主要靠 max_turns 截断。

**口述答案（90 秒）：**
> "PaiCLI 的记忆分四层。
>
> 第一层**项目记忆**：项目根目录的 `PAI.md` 或 `.paicli/PAI.md`，以及本地私有的 `PAI.local.md`。启动时 PromptAssembler 读取，截取前 4000 字符塞进 system prompt，放静态约定。
>
> 第二层**长期记忆**：`MemoryManager` 用 SQLite 存到 `~/.paicli/memory.db`，按 `scope`（项目路径）隔离。支持 save/list/search/clear，PromptAssembler 每次取最多 8 条塞进 prompt。适合存用户偏好、项目关键决策。
>
> 第三层**短期对话历史**：`Agent` 里维护 history 列表，每次运行最多 20 轮，超过就结束，这是最直接的 context 截断。
>
> 第四层**快照**：`SnapshotService` 每次运行前后复制整个项目目录到 `~/.paicli/snapshots/`，用于 `/restore` 回滚，防止 Agent 改坏代码。
>
> 配置文件里还有 context_compression、compression_threshold、token_budget_mode 等参数，但当前代码主要靠 max_turns 控制。"

**面试金句：**
> "PaiCLI 的记忆设计是'文件约定 + 长期记忆库 + 短期历史 + 快照回滚'四层。"

**可能追问：**
- 长期记忆怎么避免越塞越多？答：取最近 N 条，或按关键词 search 只召回相关记忆。
- context 满了除了截断还能怎么做？答：摘要压缩、关键信息抽取、RAG 外挂历史。

---

### G1：PaiCLI-Python 的工具注册与发现机制

**一句话定位：**
启动时 built-in + MCP 两路合并到一个 Registry 字典；运行时把工具定义喂给 LLM；执行时按读写分离做并发和审批控制。

**口述答案（90 秒）：**
> "PaiCLI 的工具系统分三层。
>
> 第一层**注册**：`bootstrap.py` 里的 `build_tool_registry` 创建一个 `ToolRegistry`（底层是字典）。先调用 `get_builtin_tools()` 注册 read_file、write_file、bash、web_search 等内置工具；如果开了 MCP，再通过 `McpClientManager` 加载外部工具注册进来。
>
> 第二层**暴露给 LLM**：`repl.py` 启动时把工具名列表塞进系统 prompt；`query.py` 每次调 LLM 前调用 `tool_registry.definitions()` 生成 OpenAI 格式的 tools 参数传过去。
>
> 第三层**执行控制**：LLM 返回 tool_calls 后，`ToolExecutor.execute_all` 处理。读工具并发执行（比如同时 read_file 多个文件），写工具严格串行防止文件冲突，高危工具如 bash 需要 HITL 审批，非只读工具执行后写审计日志。
>
> 对比 PaiAgent，PaiCLI 更强调本地 CLI 的权限、并发和审批。"

**面试金句：**
> "PaiCLI 的工具系统不是简单地让 LLM 调用函数，而是按读写分离做并发控制、按危险等级做审批、按审计做可追溯。"

**可能追问：**
- MCP 工具怎么加进来的？答：配置 MCP Server，启动时 `McpClientManager` 连接并拉取工具定义。
- 工具找不到怎么办？答：`ToolExecutor` 返回错误结果，告诉 LLM 可用工具列表。

---

### Q4：ReAct vs Plan-and-Execute 的区别？

**一句话：**
ReAct 是"边想边做"，每步根据观察调整；Plan-and-Execute 是"先计划再执行"，适合多步骤任务。

**口述答案：**
> "ReAct 是 think → act → observe 的循环，LLM 每步根据工具返回的结果决定下一步，适合需要灵活调整的任务，比如错误重试或动态工具选择。
>
> Plan-and-Execute 是先生成一个完整任务计划，再按依赖图执行。Planner 输出 DAG，Worker 并行执行，Reviewer 检查结果。适合步骤明确、可以预先分解的任务，比如'先查天气，再查路线，再生成报告'。
>
> PaiCLI-Python 同时支持两种模式，我一般在需要强可控性时用 Plan-and-Execute，在需要灵活推理时用 ReAct。"

**可能追问：**
- Plan-and-Execute 的最大风险？答：计划一旦生成，后期环境变化需要 replan，否则执行僵化。
- ReAct 的最大风险？答：容易陷入循环或 token 耗尽，需要 max_steps 限制和退出条件。

---

### Q5：Context 窗口快满了怎么办？

**口述答案：**
> "几种策略：
> 1. **短期记忆截断**：保留最近 N 轮对话，丢弃早期消息。
> 2. **长期记忆摘要**：把早期对话压缩成摘要，替代原始消息。
> 3. **关键信息抽取**：用 LLM 提取用户偏好、事实，存到长期记忆库，跨会话复用。
> 4. **RAG 外挂**：把历史文档、知识库放到向量检索里，不全部塞进 context。
> 5. **工具结果压缩**：比如搜索结果只取 top-k 摘要，不塞全文。
>
> PaiCLI-Python 里我们用快照 + 长期记忆结合：Run 前后自动快照，关键事实持久化到 SQLite，context 里只保留必要信息。"

---

### Q6：非科班非算法，为什么能做 Agent 后端？

**口述答案：**
> "Agent 后端不是算法岗，重点是工程落地能力。我的优势有三点：
> 1. **双栈工程能力**：Python FastAPI 异步栈和 Java Spring Cloud 微服务都独立交付过真实项目。
> 2. **AI 工程实践**：PaiAgent、PaiCLI-Python、PaiSmart 三个项目完整覆盖了 ReAct、Multi-Agent、MCP、RAG、Memory 等 Agent 生产化技术栈。
> 3. **业务视角**：两年零售运营经验，做过从销售、供应链到库存的全链路流程优化，这种流程思维对设计 Agent 工作流特别重要——我知道边界条件在哪、异常怎么处理。
>
> 所以我不跟算法岗拼论文，我跟工程岗拼 Agent 系统的落地能力。"

---

## 四、自我介绍（2 分钟版）

> 面试官你好，我叫曾文。
>
> 我有两条线——一条技术线，一条业务线。
>
> 技术上，我是 Python + Java 双栈后端。Python 端用 FastAPI + SQLAlchemy 2.0 异步栈做过真实上线的家装分销系统，涉及多角色权限、线索状态机、微信支付资金一致性。Java 端覆盖 Spring 全家桶，微服务治理、分布式事务、消息队列都落地过。
>
> 最近一年我重点投入 AI Agent 方向。我自研了一个企业级 Agent 编排平台 PaiAgent，实现了完整的 ReAct Agent 运行时、Tool Registry 机制、LangGraph4j 状态图引擎，还设计了 Skill 技能系统。前端用 ReactFlow 做了可视化工作流编辑器。
>
> 同时在我负责的公司项目里，我也引入了 AI 能力——为企微机器人设计了一个 Data Chart Router 层，对中文查询做意图匹配，直读数据库生成图表返回，绕过 LLM 降低了延迟和幻觉。
>
> 业务线上，我在博士眼镜做了两年零售运营，从销售到供应链到库存都跑过一遍。这段经历让我对'流程'有非常具象的理解——做 Agent 工作流引擎的时候，我比纯技术出身的同学更清楚工作流应该怎么设计、边界条件在哪。
>
> 我的优势就是把后端的工程落地能力 + AI Agent 的核心原理理解 + 业务的流程思维结合在一起。我相信自己能为团队带来即战力。
>
> 谢谢。

---

## 五、面试前 Checklist

- [ ] 自我介绍能 2 分钟流利说完，不卡顿
- [ ] 3 个核心项目（PaiAgent / PaiCLI-Python / 大宅门）各准备 1 个 STAR 故事
- [ ] PaiAgent 双引擎能讲清楚 DAG vs LangGraph 的选型
- [ ] 数据图表直查能说清楚规则 vs LLM 的分层
- [ ] 能解释 ReAct loop 的完整流程和退出条件
- [ ] 能解释 MCP 和 Function Calling 的区别
- [ ] 能解释 RAG 全流程：文档解析 → Chunk → Embedding → 检索 → 生成
- [ ] 能解释 PaiAgent / PaiCLI-Python 中记忆系统的设计
- [ ] 准备一个"你最大的技术挑战"故事
- [ ] 准备一个"你为什么离开上一家公司"的合理回答

---

## 六、待补充内容

以下题目后续演练后补充：
- 线索状态机 11 位标记设计（大宅门）
- PaiCLI-Python 的 MCP / RAG / Memory 细节
- PaiSmart 的 RAG Chunk 策略和 Embedding 选型
