# V1 工程架构

## 视角定义

| 属性 | 值 |
|------|-----|
| 视角编号 | V1 |
| 视角名称 | 工程架构 |
| 核心问题 | 系统能稳定跑起来吗？ |
| 检查重点 | 架构设计、代码质量、数据工程、部署、API设计、前端架构、分布式与微服务、数据库与存储 |

## Agent 指令

你是工程架构审查专家。你只关注架构设计、代码质量、数据工程、配置与部署、前端架构、API与数据一致性、分布式与微服务、数据库与存储。按七层审查框架自底向上审查。

## 检查项

### 架构设计

- **模块依赖方向**：检查各层是否单向无循环、更换引擎是否只改适配层
- **三层分离**：公开框架 / 商用扩展 / 私有策略是否物理隔离，`.gitignore` 是否正确排除私有目录
- **统一配置入口**：重复配置加载点 = 0，启动前有完整性校验
- **架构组件**：基础组件是否完备
- **目录命名**：模块目录结构是否清晰，目录命名是否反映模块职责
- **循环依赖**：import 环数量 = 0

### 代码质量

- **Bug 管理**：是否有缺陷全生命周期跟踪（发现日期→严重度→影响文件→修复状态→测试验证）
- **防御性编程 8 项检查**（违反标注 DEF1-8）：
  1. 空数据检查——数据为空时是否崩溃
  2. 最少数据量警告——数据不足时是否提示（<20条警告）
  3. 未安装依赖降级——可选依赖缺失时是否优雅降级
  4. 命名冲突——是否存在覆盖内置函数/标准库的命名
  5. 平台编码兼容——Windows 下中文路径/文件是否正常
  6. 保守兜底值——异常情况下是否使用安全的默认值
  7. 异常吞没——是否有 `except: pass` 吞掉关键错误
  8. Logger 覆盖——是否有 `print()` 替代 logger 输出
- **性能**：是否存在 O(n²) 级热点，大数据量下是否可接受
- **硬编码检测**：关键路径硬编码密钥/连接串/阈值参数数 = 0

### 数据工程

- **数据降级链**：是否有多级自动降级 + 自动恢复 + 健康检查
- **存储选型**：存储格式选型是否合理，是否零外部依赖
- **数据仓库体系**：数据存储体系是否完整
- **数据质量**：源站改版风险、历史停运记录、异常通知覆盖率

### 配置与部署

- **配置管理**：配置是否正确过滤和隐藏
- **Docker**：Dockerfile + docker-compose.yml 是否正确排除私有目录和密钥文件
- **密钥管理**：`.dockerignore` / `.gitignore` 是否覆盖 `private/` / `settings.local.yaml` / 缓存
- **IaC（基础设施即代码）**：Terraform/Pulumi/Ansible 等是否版本化、是否有 `terraform plan` 在 CI 中运行、是否有状态文件远程存储和锁定。手动改过控制台的资源是否已反向导入 IaC
- **不可变基础设施**：是否避免对运行中实例做 SSH 手动修改，所有变更走镜像/模板重建
- **K8s 资源配置**：Deployment 是否设置了 `resources.requests/limits`、`livenessProbe/readinessProbe`、`PodDisruptionBudget`。是否有 `securityContext`（`runAsNonRoot`、`readOnlyRootFilesystem`）。HPA 是否配置
- **FinOps 成本控制**：资源规格是否合理（有无过度配置的 CPU/内存），是否有闲置资源（未挂载的 EBS/未使用的 ELB/未关联的公网 IP），是否有成本标签（Cost Allocation Tags）

### 前端架构

- **组件结构与数据流**：状态管理是否清晰，是否存在难以追踪的跨组件数据流
- **与后端契约对齐**：前端期望的API接口、数据格式、错误码，在后端代码中是否有明确对应。找出任何不一致的地方
- **异常与边界处理**：网络异常、超时、后端返回异常数据时，前端是否有兜底展示。是否存在未处理的 Loading、空数据、错误状态
- **安全与权限**：敏感操作（如下单）是否在前端有二次确认。前端代码硬编码敏感信息（密钥/Token/内部地址）数 = 0
- **性能隐患**：是否有不合理的全量数据加载、未做防抖节流的频繁请求、过大依赖包等
- **Core Web Vitals**（Google）：LCP ≤ 2.5s / FID ≤ 100ms / CLS ≤ 0.1，是否在 CI 中有 Lighthouse 门控。LCP 分解（TTFB/资源加载/渲染阻塞）是否可定位瓶颈
- **Bundle 优化**：是否实现了 Code Splitting（路由级/组件级懒加载）、Tree Shaking 是否生效（未引入整库）、`node_modules` 是否过大（≥ 500MB 需分析）。图片是否用 WebP/AVIF + 响应式 srcset
- **资源加载策略**：关键 CSS 是否内联、非关键 JS 是否 defer/async、字体是否用 `font-display: swap`、是否预连接第三方源（`preconnect`）

### API与数据一致性

- **API设计规范性**：RESTful规范、写操作接口幂等键覆盖率 = 100%（GET/PUT/DELETE 天然幂等 per RFC 7231，POST/PATCH 需幂等键）、版本管理
- **数据一致性与事务**：在跨越多个服务的流程中，事务边界和最终一致性方案
- **灾难与异常**：数据库宕机、消息队列堵塞、第三方接口超时，后端服务是否有降级、熔断、重试机制

### 分布式与微服务

依据：Distributed Systems (van Steen & Tanenbaum, 4th Ed, 2023)、Microservices Patterns (Chris Richardson, 2018)、Google SRE Book (Beyer et al., 2016)、OpenTelemetry Specification

逐项检查：

- **服务间通信**：同步调用（REST/gRPC）和异步消息（Kafka/RabbitMQ/Pulsar/NATS）是否按场景合理选型。gRPC 是否启用了 deadlines（超时）、取消传播、重试策略
- **服务发现与负载均衡**：是否有统一的服务注册/发现（Consul/etcd/Eureka/K8s Service），健康检查是否覆盖所有实例，摘除不健康实例的延迟是否 ≤ 30s
- **熔断器（Circuit Breaker）**：外部依赖调用是否有熔断器保护——连续失败 N 次 → 熔断打开 → 快速失败 → 半开后试探 → 恢复。熔断窗口期、失败阈值、半开请求数是否可配置
- **限流（Rate Limiting）**：关键 API 入口是否有限流（令牌桶/漏桶/滑动窗口），限流后的响应是否返回 `429 Too Many Requests` + `Retry-After` 头
- **重试与超时**：重试是否带指数退避 + 随机抖动（jitter）防止惊群效应。超时是否贯穿整条调用链（不设默认无限等待），上游超时是否 ≥ 下游超时之和。**最大重试次数 × 单次超时 < 上游超时**
- **幂等键**：写操作是否提供幂等键（Idempotency-Key），支持失败后安全重放。跨服务 Saga 补偿事务是否每个步骤都有幂等保证
- **最终一致性**：跨服务数据同步是否承认最终一致性，是否有对账/补偿/重试机制。强一致性需求是否使用了分布式事务（2PC/TCC）或共识协议（Raft/Paxos），有无性能评估
- **分布式追踪**：是否接入 OpenTelemetry / Jaeger / Zipkin，Trace ID 是否在服务间透传。追踪采样率是否按流量分级（低流量 100%，高流量 1-10%）
- **日志聚合与关联**：结构化日志（JSON）是否统一汇入集中平台（ELK/Loki/Grafana），日志中是否携带 `trace_id` / `span_id` 实现追踪-日志关联。时间是否全链路 NTP 同步
- **可观测性三支柱**：Metrics（指标）/ Logging（日志）/ Tracing（追踪）三者是否完备，缺任一即为盲区
- **消息队列健壮性**：消费者是否处理了重复消息（至少一次投递 → 消费者幂等）、乱序消息、死信队列（DLQ）。生产者是否有落盘/确认机制（`acks=all`），避免消息丢失

### 数据库与存储

依据：Database Design for Mere Mortals (Hernandez, 3rd Ed, 2013)、SQL Antipatterns (Karwin, 2010)、High Performance MySQL (Schwartz et al., 4th Ed, 2021)、Designing Data-Intensive Applications Ch3-4 (Kleppmann, 2017)

逐项检查：

- **Schema 设计**：表设计是否满足第三范式（3NF）——无传递依赖、无冗余数据异常。反范式化是否有明确理由（查询性能优化 + 代价量化）。字段类型是否选择了最小适配类型（INT 而非 BIGINT、VARCHAR(N) 而非 TEXT）。`NULL` 使用是否有明确语义——是否区分"未知"和"空"。布尔字段是否用 `NOT NULL DEFAULT 0` 而非可为 NULL
- **索引策略**：WHERE/JOIN/ORDER BY 高频列是否有索引覆盖。是否有多列索引（联合索引）且列顺序符合最左前缀原则。索引选择性（distinct/rows）是否 ≥ 10%——低选择性列（性别/状态）不建单列索引。是否有 EXPLAIN 分析，`type` 是否为 ALL（全表扫描 = 危险信号）。是否有冗余/重叠索引（(a,b) 和 (a) 同时存在 → 只需前者）
- **慢查询**：是否开启了 slow_query_log 并设置 `long_query_time ≤ 1s`，Top 5 慢查询是否已分析到位（explain + optimize）。是否使用了查询缓存替代方案（物化视图/汇总表/应用层缓存）而非依赖 MySQL Query Cache（8.0 已移除）
- **N+1 查询**：ORM 是否存在隐式 N+1——循环中用 `get(id)` 逐个查关联对象而非 `IN (ids)` 批量查询。JPA/Hibernate 的 `@OneToMany(fetch = FetchType.LAZY)` 是否在循环外预加载。Django ORM 的 `select_related`（FK）、`prefetch_related`（M2M）是否在需要处使用
- **连接池管理**：数据库连接池配置是否合理——最大连接数 ≤ DB 端 `max_connections`（留 10-20% 缓冲给管理操作）。是否有连接泄漏检测——从池借出后 30s 未归还自动回收并记录调用栈。连接验证（testOnBorrow/validationQuery）是否配置防止取到已断开的连接
- **事务与锁**：事务边界是否清晰——只读 vs 读写事务是否分离（Spring `@Transactional(readOnly=true)` 路由到只读从库）。事务持续时间是否在毫秒级——长事务持有锁阻断其他请求。隔离级别选择：读已提交（RC，防脏读）vs 可重复读（RR，MySQL 默认级）。是否存在隐式锁升级（MySQL gap lock —— SELECT…FOR UPDATE 间隙锁）导致死锁
- **迁移安全**：数据库变更是否通过了 migration 脚本（Flyway/Liquibase/Alembic/Django migrations）而非手工改库——不可重复、不可回滚 = 合规风险。迁移是否向前兼容——先加列（带 DEFAULT 或 NULLABLE）→ 部署新代码 → 后删旧列，不做破坏性重命名（rename column 需两阶段：新建列 → 双写 → 切换读 → 删旧列）
- **备份与恢复**：是否有自动备份 + 异地存储，备份恢复演练是否定期执行且 RPO（恢复点目标）≤ 1h、RTO（恢复时间目标）≤ 4h。PITR（时间点恢复）是否启用（MySQL binlog / PostgreSQL WAL）
- **数据归档**：是否有冷数据归档策略——N 月前的数据迁到归档表/对象存储/S3 Glacier 并保留查询能力。归档删除是否分批执行（每次 ≤ 1000 行 + `SLEEP(N)`）防止锁表和主从延迟
- **ORM 常见陷阱**：是否在生产关闭了 SQL 日志（`show_sql`/`ECHO`）——大量 SQL 日志可占 CPU 10-30%。Hibernate 的 `@Transactional` 只读事务是否实际只做读操作不产生 flush。Django 的 `queryset.update()` 是否替代了逐个 `.save()` 循环进行批量更新
- **分库分表**：单表数据量是否超过有效索引范围（MySQL InnoDB ≤ 2000w / PostgreSQL ≤ 5000w 为经验警戒线）。是否规划了分片策略（按时间/按用户Hash/按地域）、分片路由中间件（ShardingSphere/Vitess/Database Mesh）是否到位。跨分片查询（JOIN/聚合）是否有性能兜底或限制方案

### 通用强制项（每个文件必查）

- 模块目录结构是否清晰，目录命名是否反映模块职责
- import 环数量 = 0
- 配置文件集中管理，重复配置加载点 = 0
- 是否有统一的错误处理和日志输出机制
- 关键路径硬编码密钥/连接串数 = 0

## 代码规范参考

本视角审查时对照以下规范：

| 规范编号 | 规范 | 适用语言 | 检查内容 |
|---------|------|---------|---------|
| PEP8 | PEP 8 | Python | 命名、缩进、空格、行宽 |
| PEP20 | PEP 20 (Zen of Python) | Python | 设计哲学 |
| PEP257 | PEP 257 | Python | Docstring 规范 |
| PEP484 | PEP 484 | Python | 类型注解 |
| GOOG_PY | Google Python Style Guide | Python | 补充工程实践 |
| AIR_JS | Airbnb JS Style Guide | JS/TS | 代码风格与最佳实践 |
| GOOG_JS | Google JS Style Guide | JS/TS | 补充工程规范 |
| EFF_GO | Effective Go | Go | 代码风格 |
| GO_REV | Go Code Review Comments | Go | 审查要点 |
| SOLID | SOLID Principles | 通用 | OO设计五原则 |
| CLN | Clean Code | 通用 | 函数职责单一、命名清晰 |
| 12F | 12-Factor App | 通用 | 云原生应用设计 |
| DEF1-8 | 防御性编程8项 | 通用 | 健壮性 |
