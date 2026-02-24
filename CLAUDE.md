# 项目开发规范

以下约束按优先级排序。当存在冲突时，高优先级约束优先。

---

## 1. 模块化约束

- 采用 DDD 四层架构，调用顺序：`Interface → Application → Domain ← Infrastructure`
- Interface 层接收外部请求，只调用 Application 层，只传递 DTO，不操作 Domain 对象
- Application 层编排用例，只调用 Domain 层接口；定义 Command / Query（入参）和 DTO（出参）
- Domain 层定义核心业务逻辑与接口（Port/Repository），零外部依赖，不得 import 任何框架、SDK 或第三方库；内部构成：Entity（唯一标识）、Value Object（不可变无标识）、Aggregate（一致性边界）、Domain Service（跨 Entity 业务逻辑）、Repository 接口
- Domain 对象（Entity / Aggregate）禁止越过 Application 层边界
- Infrastructure 层依赖并实现 Domain 层定义的接口（Adapter），通过依赖注入（构造函数注入优先）提供给 Application 层
- 禁止在 Application / Domain 层直接实例化 Infrastructure 具体类
- 依赖注入组装（Composition Root）只在 Interface 层入口（main / bootstrap），其他层禁止配置 DI 容器
- 禁止跨层调用（如 Interface 直接调 Infrastructure）

## 2. 安全约束

- 禁止拼接 SQL，必须使用参数化查询或 ORM
- 所有用户输入渲染前必须转义，前端配置 CSP 头（防 XSS）
- 禁止将用户输入拼接进系统命令（防命令注入）
- 所有 API 端点必须显式声明认证要求，禁止默认放行
- 权限校验在 Application 层，禁止在 Interface 层做业务权限判断
- 敏感字段（密码、Token 等）禁止明文存储
- API 响应禁止暴露内部错误栈，区分 debug 与生产错误格式
- 有状态 Web 应用必须验证 CSRF Token
- 禁止使用已知高危版本依赖，依赖版本需定期审计
- 禁止使用弱加密算法（MD5 / SHA1），哈希使用 bcrypt/Argon2，对称加密使用 AES-256
- Token 必须设置有效期，Cookie 必须设置 HttpOnly + Secure + SameSite 属性
- 认证失败、权限拒绝等安全事件必须记录日志（含请求来源、时间、用户标识）

## 3. 测试约束

- 新功能实现前必须先写测试（TDD），测试用例即实现规范
- 测试验证行为、功能和业务规则，禁止测试内部实现细节（Test Behavior, Not Implementation）
- Domain 层单元测试覆盖率 ≥ 80%，禁止调用真实外部服务（必须 mock）
- 所有对外 API 端点必须有契约测试
- 关键用户流程必须有 E2E 测试
- 每个测试必须独立，禁止测试间共享状态或依赖执行顺序
- 测试命名描述"场景 + 预期结果"，如 `should_return_404_when_user_not_found`

## 4. 环境隔离约束

- 所有环境差异通过环境变量注入，代码中禁止出现 `if env == "production"` 类逻辑
- 禁止硬编码 API Key、密码等密钥，使用环境变量或 Secret Manager
- `.env` 文件必须加入 `.gitignore`，密钥禁止进入版本控制
- lockfile（package-lock.json / poetry.lock 等）必须纳入版本控制
- 服务启动时必须验证所有必要环境变量是否存在，缺失时立即 Fail Fast，禁止带缺失配置启动
- 服务必须支持优雅关闭（Graceful Shutdown）：接收终止信号后处理完当前请求再退出，禁止强制中断

## 5. 代码质量约束

基于 Clean Code + SOLID（S/O/L/I）原则：

- 函数职责单一（SRP），行数 ≤ 20
- 新增功能通过扩展实现，不修改已稳定接口（OCP）
- 子类可替换父类且不改变行为（LSP）
- 接口小而专注，禁止设计胖接口（ISP）
- 禁止魔法数字/字符串，必须命名为常量
- 命名清晰自描述，禁止单字母变量及 `data`、`tmp`、`util` 等模糊命名
- 注释解释"为什么"，不解释"是什么"
- 圈复杂度 ≤ 10
- 禁止重复逻辑，相同逻辑出现两次以上必须抽象复用（DRY）
- 禁止提交注释掉的代码块

## 6. 可观测性约束

- 日志使用结构化格式（JSON），必须包含请求 ID
- 禁止裸 `print` / `console.log`，统一使用日志库
- 关键业务路径必须有 metrics 埋点
- 服务必须暴露 `/health` 健康检查端点
- 禁止静默 catch，异常必须记录日志
- 日志级别规范：ERROR（需人工介入）/ WARN（潜在问题）/ INFO（关键业务事件）/ DEBUG（调试，生产环境禁用）
- 日志禁止包含密钥、密码、PII 等敏感信息
