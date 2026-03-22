# 私有化部署方案

## 部署目标

- 支持企业内网环境，最小化外网依赖
- 支持按租户规模进行横向扩展
- 控制面、数据面、任务面分离部署
- 满足高可用、审计、备份、容灾要求

## 推荐逻辑拓扑

### 网关层

- API Gateway / Ingress
- WAF 或企业安全网关
- SSO 接入与统一认证代理

### 应用层

- FastAPI 平台服务
- 授权服务
- 配置中心
- 控制台前端静态资源服务

### 任务层

- 文档导入 Worker
- 解析/OCR Worker
- Embedding/索引 Worker
- 评测 Worker

### 检索与生成层

- 检索服务
- 重排服务
- 生成服务
- Provider 路由服务

### 存储层

- PostgreSQL 主从或高可用集群
- Redis 高可用
- MQ 集群
- 对象存储
- Elasticsearch/OpenSearch 集群
- Milvus 或 pgvector

### 观测层

- Prometheus
- Grafana
- Loki/ELK
- OpenTelemetry Collector

## 部署建议

- 控制台和开放 API 可共享网关，但使用不同路径和策略
- 文档处理 Worker 与在线问答服务物理隔离，避免资源争用
- 大模型推理建议独立资源池，支持 GPU 和 CPU 混部
- 向量库和搜索引擎优先独立部署，避免与主库争抢 IO

## 高可用与容灾

- 应用服务无状态化，使用多副本部署
- PostgreSQL 开启备份、WAL 归档和主备切换
- 对象存储启用版本控制和跨存储池备份
- MQ 启用持久化和消费重试队列
- 关键索引支持重建和双版本切换

## 容量规划基线

- 文档规模按“文档数、总页数、总切片数、总向量数”评估
- 在线服务按 `QPS`、平均上下文长度、模型响应时间评估
- Worker 池按导入高峰、重建周期和单文档处理时长评估
- 存储容量至少预留原始文档与中间产物的 2 到 3 倍空间

## 可替换基础依赖

- 消息中间件：Kafka 或 RabbitMQ
- 全文检索：Elasticsearch 或 OpenSearch
- 向量能力：Milvus 或 PostgreSQL + pgvector
- 对象存储：MinIO 或企业已有 S3 兼容存储
