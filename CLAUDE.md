# 项目开发规范

> 本规范适用于内部微服务，不包含前端、API Gateway 或对外暴露服务的场景。
> 以下约束按优先级排序。当存在冲突时，高优先级约束优先。

---

## 1. 安全约束

- 禁止拼接 SQL，必须使用参数化查询或 ORM
- 禁止将外部输入拼接进系统命令（防命令注入）
- 敏感字段（密码等）禁止明文存储
- API 响应禁止暴露内部错误栈，区分 debug 与生产错误格式
- 禁止使用已知高危版本依赖，依赖版本需定期审计
- 禁止使用弱加密算法（MD5 / SHA1），哈希使用 bcrypt/Argon2，对称加密使用 AES-256
- 安全事件必须记录日志（含请求来源、时间、调用方标识）

## 2. 模块化约束

采用 DDD 四层架构，调用顺序：`Interface → Application → Domain ← Infrastructure`

**各层职责与判断规则：**

| 层 | 职责 | LLM 判断规则 |
|---|---|---|
| Interface | 接收请求、参数校验、序列化 | 只调用 Application 层；只传 DTO，不传 Domain 对象 |
| Application | 编排用例，定义 Command/Query（入参）和 DTO（出参） | 自身不定义业务规则，只编排 Domain 对象完成用例；只调用 Domain 接口 |
| Domain | 核心业务规则，零外部依赖 | 有业务规则判断时放此层；禁止 import 任何框架/SDK/第三方库 |
| Infrastructure | 实现 Domain 定义的接口 | 数据库、消息队列、外部 HTTP 等所有 I/O 实现放此层 |

**具体判断规则：**

- 业务规则（如"金额不能为负"、"状态机流转"）→ 放 Domain Entity 或 Domain Service
- 跨多个 Aggregate 的协调逻辑 → 放 Application Use Case；业务规则本身仍定义在 Domain 对象内
- Repository 接口 → 定义在 Domain 层；实现 → 放 Infrastructure 层
- 有唯一标识且有生命周期 → 建 Entity；无标识且不可变 → 建 Value Object
- 跨 Entity 的业务逻辑且不属于任何单一 Entity → 建 Domain Service
- Aggregate 之间禁止直接持有对象引用，只能通过 ID 引用；跨 Aggregate 的数据加载由 Application 层协调
- Domain 对象（Entity / Aggregate）禁止越过 Application 层边界
- 禁止在 Application / Domain 层直接实例化 Infrastructure 具体类
- 依赖注入组装（Composition Root）只在 Interface 层入口（main / bootstrap），其他层禁止配置 DI 容器
- 禁止跨层调用（如 Interface 直接调 Infrastructure）
- 异常跨层转换规则：
  - Domain 层：抛出领域异常（DomainException），只表达业务含义，不涉及 HTTP 或框架概念
  - Application 层：捕获 DomainException 后按需包装为业务异常；禁止将 Domain 异常直接透传至 Interface 层
  - Interface 层：统一捕获异常并映射为 HTTP 状态码；禁止在 Domain / Application 层出现任何 HTTP 概念

## 3. 测试约束

- 新功能实现前必须先写测试（TDD），测试用例即实现规范
- 测试只验证可观测的最终结果（返回值、存储状态、发布事件），禁止断言内部调用顺序、中间对象结构或私有状态
- 每个测试必须独立，禁止测试间共享状态或依赖执行顺序
- 测试命名描述"场景 + 预期结果"，如 `should_reject_order_when_credit_limit_exceeded`

**各层测试规则：**

| 层 | 是否写测试 | 测什么 |
|---|---|---|
| Domain | 必须写，覆盖率 ≥ 80% | 业务规则、状态流转、Value Object 校验、Domain Service 逻辑 |
| Application | 必须写 | Use Case 的可观测结果：返回值、存储状态、发布事件 |
| Infrastructure | 不写单元测试 | 数据库/HTTP 等 I/O 不做单元测试 |
| Interface | 不写 | 纯框架路由，无业务逻辑 |

**禁止使用 Mock 框架（Mockito / unittest.mock 等）：**

Mock + verify 断言的是"调用了哪个方法"，属于实现细节，重构时必然破坏测试。统一使用 **In-memory Fake** 替代：

```python
# ❌ 禁止：测实现细节，重构即失败
mock_repo.save.assert_called_once_with(order)

# ✓ 必须：测可观测结果，重构不影响
repo = InMemoryOrderRepository()
use_case.execute(cmd)
assert repo.find_by_id(result.order_id) is not None
```

每个 Repository / 外部服务接口须提供对应的 In-memory Fake 实现，与生产实现同级维护。

## 4. 环境隔离约束

- 所有环境差异通过环境变量注入，代码中禁止出现 `if env == "production"` 类逻辑
- 禁止硬编码 API Key、密码等密钥，使用环境变量或 Secret Manager
- `.env` 文件必须加入 `.gitignore`，密钥禁止进入版本控制
- lockfile（package-lock.json / poetry.lock 等）必须纳入版本控制
- 服务启动时必须通过统一配置验证模块校验所有必要环境变量，缺失时立即 Fail Fast，禁止带缺失配置启动
- 服务必须支持优雅关闭（Graceful Shutdown）：接收终止信号后处理完当前请求再退出，禁止强制中断

## 5. 代码质量约束

基于 Clean Code + SOLID（S/O/I）原则：

- 函数职责单一（SRP），行数 ≤ 20
- 新增功能通过扩展实现，不修改已稳定接口（OCP）
- 接口小而专注，禁止设计胖接口（ISP）
- 禁止魔法数字/字符串，必须命名为常量
- 命名清晰自描述，禁止单字母变量及 `data`、`tmp`、`util` 等模糊命名
- 注释解释"为什么"，不解释"是什么"
- 圈复杂度 ≤ 10
- 禁止重复逻辑，相同逻辑出现两次以上必须抽象复用（DRY）
- 禁止提交注释掉的代码块

## 6. 可观测性约束

- 日志使用结构化格式（JSON），每条日志必须包含 `trace_id`
- `trace_id` 从上游请求头 `X-Request-ID` 读取并贯穿整个调用链；调用下游服务时必须透传该请求头
- 禁止裸 `print` / `console.log`，统一使用日志库
- 关键业务路径必须有 metrics 埋点
- 服务必须暴露 `/health` 健康检查端点
- 禁止静默 catch，异常必须记录日志
- 日志级别规范：ERROR（需人工介入）/ WARN（潜在问题）/ INFO（关键业务事件）/ DEBUG（调试，生产环境禁用）
- 日志禁止包含密钥、密码、PII 等敏感信息
