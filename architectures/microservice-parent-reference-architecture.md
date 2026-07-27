# 微服务 Parent 工程参考架构

## 适用场景

适用于 Java / Spring Boot / Spring Cloud 类多服务项目，尤其是多个业务系统共享一套基础能力时。

典型场景：

- 多个业务服务需要统一依赖版本、公共工具、接口模型和基础组件。
- 新项目希望快速落地一套可复用的微服务骨架。
- 需要把项目经验沉淀到 parent 工程，减少重复造轮子。

## 设计目标

- 统一依赖版本和构建方式。
- 沉淀跨项目公共能力。
- 保持业务服务轻量，避免每个服务重复维护基础代码。
- 明确 parent 边界，避免 parent 变成业务大杂烩。
- 让其他项目可按模块选择性复用。

## 推荐目录结构

```text
parent-project/
├── pom.xml
├── common-bom/                 # 依赖版本约束，可选
├── common-core/                # 基础模型、常量、异常、返回结构
├── common-web/                 # Web 通用能力
├── common-security/            # 认证、鉴权、数据权限基础能力
├── common-db/                  # MyBatis/JPA、分页、审计字段、数据源配置
├── common-redis/               # Redis 工具、缓存 key 规范、分布式锁
├── common-mq/                  # MQ 消息模型、生产消费基础封装
├── common-log/                 # 日志、链路追踪、操作日志
├── common-file/                # 文件上传、下载、对象存储抽象
├── common-job/                 # 定时任务、批处理基础能力
├── common-api/                 # 跨服务 Feign/API DTO 定义
├── common-test/                # 测试基类、Mock 工具、集成测试辅助
└── services/
    ├── user-service/
    ├── monitor-service/
    └── asset-service/
```

实际项目可按体量裁剪，不必一次性创建所有模块。

## Parent 的核心职责

### 依赖与构建

Parent `pom.xml` 建议承担：

- Java 版本。
- Spring Boot / Spring Cloud 版本。
- Maven 插件版本。
- 三方依赖版本管理。
- 编译、测试、打包的基础配置。

不建议在 parent `pom.xml` 中直接堆业务依赖。业务服务按需引入公共模块。

### 公共模型

可放入 `common-core`：

- 统一响应结构，例如 `Result`、`PageResult`。
- 分页请求和分页返回模型。
- 基础异常与错误码。
- 通用常量、枚举基础接口。
- 时间、金额、树结构、字典等通用 DTO。
- 脱敏、校验、转换等纯工具类。

注意：业务强绑定的枚举、表字段、状态流不要放进 `common-core`。

### Web 基础能力

可放入 `common-web`：

- 全局异常处理。
- 参数校验错误格式化。
- 请求上下文。
- Web 拦截器基础封装。
- Jackson 时间格式、枚举序列化配置。
- Swagger / OpenAPI 基础配置。
- Controller 返回值统一处理。

### 安全与权限

可放入 `common-security`：

- Token 解析。
- 当前用户上下文。
- 权限注解和切面基础能力。
- 租户上下文。
- 数据权限过滤基础接口。
- 脱敏注解和基础实现。

项目落地时要区分：

- 通用认证鉴权机制可以复用。
- 具体角色、菜单、按钮、数据范围规则通常属于业务系统。

### 数据库与持久化

可放入 `common-db`：

- MyBatis / MyBatis Plus 基础配置。
- 分页插件配置。
- 自动填充创建人、创建时间、更新人、更新时间。
- 逻辑删除基础约定。
- 通用 Mapper 基类或查询工具。
- 多数据源基础配置。
- SQL 日志和慢查询辅助配置。

不建议放入：

- 具体业务表 Entity。
- 具体业务 Mapper。
- 业务 SQL。

### Redis 与缓存

可放入 `common-redis`：

- RedisTemplate 序列化配置。
- 缓存工具。
- 分布式锁封装。
- 幂等 key 工具。
- 缓存 key 命名规范。
- TTL 常量和缓存失效辅助方法。

缓存 key 建议按业务项目定义前缀，parent 只提供拼接规范和工具。

### MQ 与事件

可放入 `common-mq`：

- 通用消息头。
- 消息唯一 id、trace id、tenant id。
- 生产者基础封装。
- 消费者幂等基础能力。
- 重试、死信、延迟消息基础配置。
- 事件模型基础接口。

不建议把具体业务 topic、queue、exchange 全部固化到 parent。业务项目可以基于公共规范单独声明。

### 日志与链路追踪

可放入 `common-log`：

- trace id 生成与透传。
- 统一日志字段。
- 操作日志注解。
- 慢接口日志。
- 第三方接口调用日志辅助方法。
- 敏感字段脱敏工具。

### 文件服务

可放入 `common-file`：

- 文件上传、下载接口抽象。
- 本地存储、对象存储适配接口。
- 文件元数据 DTO。
- 文件类型、大小、后缀校验。
- 预签名 URL 基础能力。

具体存储桶、目录规则、业务归属关系由业务服务定义。

### 定时任务与批处理

可放入 `common-job`：

- 定时任务开关约定。
- 分布式锁封装。
- 批处理分页工具。
- 任务执行摘要模型。
- 成功数、失败数、耗时日志工具。
- 单条失败继续执行的模板方法。

### 跨服务 API

可放入 `common-api` 或按领域拆分为 `user-api`、`asset-api`：

- Feign Client 接口。
- 跨服务请求 DTO。
- 跨服务响应 DTO。
- 服务间错误码约定。

原则：

- API 模块只放跨服务契约。
- 不放服务内部 Entity。
- 不暴露数据库表结构细节。

### 测试基础

可放入 `common-test`：

- 测试基类。
- Mock 用户上下文。
- Mock 租户上下文。
- Testcontainers 或嵌入式组件配置。
- 接口自测脚本公共方法。

## 不建议放入 Parent 的内容

- 具体业务流程。
- 具体业务表 Entity / Mapper / SQL。
- 具体菜单、角色、按钮权限。
- 具体第三方账号、密钥、环境地址。
- 仅单个服务使用的工具类。
- 一次性脚本和临时兼容逻辑。
- 会导致所有服务被迫引入的重依赖。

判断标准：

- 至少两个项目或多个服务稳定复用，才考虑上沉。
- 与业务语义强绑定的内容留在业务服务。
- 依赖重、变化快、不稳定的能力先放业务侧，成熟后再抽象。

## 推荐落地步骤

1. 先梳理现有服务重复代码。
2. 按能力域归类：core、web、security、db、redis、mq、file、job、api、test。
3. 优先抽取无业务语义的基础能力。
4. 为每个公共模块定义清晰依赖边界。
5. 业务服务逐个替换重复实现。
6. 补充 README、使用示例和最小验证命令。
7. 新项目接入时按需选择模块，不一次性全量引入。

## 依赖边界建议

推荐方向：

```text
common-core
  ↑
common-web / common-db / common-redis / common-log
  ↑
common-security / common-mq / common-file / common-job
  ↑
business-service
```

原则：

- `common-core` 不依赖 Web、DB、Redis、MQ。
- `common-api` 尽量只依赖 `common-core`。
- 公共模块之间避免循环依赖。
- 业务服务可以依赖公共模块，公共模块不依赖业务服务。

## 新项目推荐清单

最小可落地组合：

- `common-core`
- `common-web`
- `common-db`
- `common-log`
- `common-security`

按需扩展：

- 有缓存：增加 `common-redis`。
- 有 MQ：增加 `common-mq`。
- 有文件：增加 `common-file`。
- 有批处理：增加 `common-job`。
- 多服务调用：增加 `common-api` 或领域 API 模块。
- 重视自动化测试：增加 `common-test`。

## 验证方式

抽取 parent 公共能力后，至少验证：

- 公共模块自身能编译通过。
- 至少一个业务服务接入后能编译通过。
- 原有接口返回格式不变。
- 权限、租户、数据范围逻辑不回退。
- MQ、Redis、文件、数据库配置支持按环境覆盖。
- 新项目按 README 能完成最小启动。

## 推荐交付物

每次沉淀 parent 能力时，建议同时交付：

- 公共模块源码。
- 使用示例。
- 依赖引入示例。
- 配置项说明。
- 最小验证命令。
- 迁移说明。
- 剩余限制和不适用场景。
