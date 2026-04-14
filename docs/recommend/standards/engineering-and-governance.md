# 工程与治理规范

## 目标

本规范用于统一一期推荐平台与未来全量 Go 迁移的工程结构、接口标准、日志规范、配置治理和公共组件约束。

## Monorepo 目录规范

```text
repo/
  apps/
    recommend-bff/
    user-profile-service/
    recall-rank-service/
    promo-decision-service/
    event-collector/
    feature-pipeline/
    catalog-read-service/
  api/
    proto/
    openapi/
  pkg/
    auth/
    config/
    experiment/
    mq/
    observability/
  deploy/
    helm/
    k8s/
  docs/
    architecture/
    implementation/
    standards/
```

## 服务内部分层规范

每个服务统一使用如下结构：

```text
apps/<service-name>/
  cmd/
  internal/
    handler/
    service/
    domain/
    repository/
    types/
    config/
  etc/
  Makefile
  README.md
```

职责约束：

- `handler`
  - 协议适配层
  - 参数校验
  - 响应封装
- `service`
  - 用例编排
  - 事务边界
  - 调用 domain 与 repository
- `domain`
  - 核心业务规则
  - 推荐策略
  - 排序逻辑
  - 画像规则
- `repository`
  - MySQL
  - Redis
  - Kafka
  - ES/OpenSearch
  - 第三方依赖
- `types`
  - DTO
  - API Request/Response
  - 内部 ViewModel
- `config`
  - 配置结构体
  - 配置加载逻辑

## 命名规范

### 服务命名

- 统一使用短横线目录名，例如：
  - `recommend-bff`
  - `user-profile-service`
  - `promo-decision-service`

### 包命名

- 统一使用小写英文
- 避免缩写不清的名字
- 避免 `util`、`common` 这种无边界命名泛滥

### 配置项命名

- 配置分层以服务域名为前缀
- 例如：
  - `recommend.scene.home.cache_ttl`
  - `recommend.rank.default_strategy`
  - `promo.coupon.gray_ratio`

## API 标准

### 对外 HTTP 接口

- 全部使用 `HTTP/JSON`
- 通过 `OpenAPI` 描述
- 路径统一加版本号
- 推荐采用：
  - `/api/v1/recommend/...`
  - `/api/v1/events/...`

### 服务间接口

- 全部使用 `gRPC + proto`
- `proto` 为唯一契约来源
- 禁止手写多份重复 DTO

### 错误码规范

- 统一错误码包
- 错误码分域：
  - 网关域
  - 推荐域
  - 画像域
  - 活动域
  - 数据同步域
- 对外响应包含：
  - code
  - message
  - request_id

## OpenAPI 规范

### 必须项

- 每个接口必须有：
  - summary
  - operationId
  - tags
  - request schema
  - response schema
  - error response

### 推荐接口返回字段

所有推荐接口必须统一返回：

- request_id
- trace_id
- scene
- experiment_id
- strategy_id
- feature_version
- items

## proto 规范

### 目录建议

- `api/proto/recommend/v1`
- `api/proto/profile/v1`
- `api/proto/promo/v1`
- `api/proto/catalog/v1`
- `api/proto/event/v1`

### 约束

- 字段禁止无意义缩写
- 必须保留向后兼容意识
- 废弃字段用 `reserved`
- 时间统一使用：
  - Unix 毫秒
  - 或标准化时间字符串

## 日志规范

### 格式

- 统一输出结构化 JSON
- 禁止生产环境依赖非结构化拼接日志

### 必须字段

- timestamp
- level
- service
- env
- version
- trace_id
- request_id
- user_id
- anonymous_id
- scene
- experiment_id
- strategy_id
- message

### 推荐链路建议追加字段

- item_ids
- recall_channels
- rank_cost_ms
- profile_version
- feature_version
- fallback_reason

## OpenTelemetry 规范

### Trace

- 每个 HTTP 请求必须创建 span
- 服务间 gRPC 调用必须透传 trace 上下文
- Kafka 生产/消费必须透传 trace 关联信息

### Metrics

推荐服务统一输出：

- qps
- latency_ms
- error_rate
- fallback_rate
- cache_hit_rate
- profile_lookup_latency
- rank_latency
- event_ingest_qps
- kafka_lag

### Logs

- 日志中必须能关联 trace_id
- 推荐结果异常与回退必须可检索

## Nacos 配置规范

## 命名空间与分组

- 按环境隔离：
  - `dev`
  - `test`
  - `prod`
- 按域分组：
  - `recommend`
  - `profile`
  - `promo`
  - `infra`

### 配置分类

- 静态运行配置
  - 服务端口
  - 依赖地址
  - 超时
- 动态策略配置
  - 实验开关
  - 召回开关
  - 排序权重
  - 运营白名单
  - 灰度比例

### 禁止事项

- 敏感密钥不直接放普通业务配置
- 不把配置当数据库使用
- 不在代码里硬编码实验参数

## 配置加载顺序

建议顺序：

1. 本地默认配置
2. 环境变量覆盖
3. Nacos 动态配置覆盖

原则：

- 本地开发可运行
- 测试和生产以 Nacos 为准

## 事件规范

### 公共字段

- event_id
- event_type
- user_id
- anonymous_id
- device_id
- scene
- page
- request_id
- trace_id
- experiment_id
- strategy_id
- occurred_at
- source

### 事件版本

- 每类事件必须带 schema version
- 变更时新增版本，不直接破坏老消费者

## Redis 使用规范

- 用途：
  - 热画像缓存
  - 推荐结果缓存
  - 限流
  - 幂等
- 不作为推荐主事件流

key 设计建议：

- `profile:user:{user_id}`
- `recommend:scene:{scene}:user:{user_id}`
- `experiment:bucket:{user_id}`

## Kafka 使用规范

- topic 按业务域划分
- 事件生产必须带 trace_id
- 消费失败必须进入重试或死信队列
- 不允许业务 silently drop event

建议 topic：

- `user_behavior_events`
- `order_domain_events`
- `payment_domain_events`
- `recommend_exposure_events`
- `recommend_click_events`

## 数据库规范

- 读模型与画像库分离
- 表名与服务域一致
- 统一审计字段：
  - created_at
  - updated_at
  - deleted_at
- 不允许跨服务直接写库

## 代码质量规范

- 所有服务必须有：
  - README
  - 配置说明
  - OpenAPI 或 proto 契约
  - 基础测试
- CI 至少校验：
  - format
  - lint
  - unit test
  - contract check

## 文档规范

- 架构决策文档单独维护
- 新增服务前必须先补：
  - 服务职责
  - 数据边界
  - 接口清单
  - 观测指标
