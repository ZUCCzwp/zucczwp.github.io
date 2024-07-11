# 分布式链路追踪


# 分布式链路追踪

## 核心概念

Trace 是一个逻辑概念，表示一次（分布式）请求经过的所有局部操作（Span）构成的一条完整的有向无环图，其中所有的 Span 的 TraceId 相同。

Span 则是真实的数据实体模型，表示一次(分布式)请求过程的一个步骤或操作，代表系统中一个逻辑运行单元，Span 之间通过嵌套或者顺序排列建立因果关系。Span 数据在采集端生成，之后上报到服务端，做进一步的处理。其包含如下关键属性：

- Name：操作名称，如一个 RPC 方法的名称，一个函数名
- StartTime/EndTime：起始时间和结束时间，操作的生命周期
- ParentSpanId：父级 Span 的 ID
- Attributes：属性，一组 <K,V> 键值对构成的集合
- Event：操作期间发生的事件
- SpanContext：Span 上下文内容，通常用于在 Span 间传播，其核心字段包括 TraceId、SpanId

## 一般架构

在应用端需要通过侵入或者非侵入的方式，注入 Tracing Sdk，以跟踪、生成、传播和上报请求调用链路数据；

Collect agent 一般是在靠近应用侧的一个边缘计算层，主要用于提高 Tracing Sdk 的写性能，和减少 back-end 的计算压力；

采集的链路跟踪数据上报到后端时，首先经过 Gateway 做一个鉴权，之后进入 kafka 这样的 MQ 进行消息的缓冲存储；

在数据写入存储层之前，我们可能需要对消息队列中的数据做一些清洗和分析的操作，清洗是为了规范和适配不同的数据源上报的数据，分析通常是为了支持更高级的业务功能，比如流量统计、错误分析等，这部分通常采用flink这类的流处理框架来完成；

存储层会是服务端设计选型的一个重点，要考虑数据量级和查询场景的特点来设计选型，通常的选择包括使用 Elasticsearch、Cassandra、或 Clickhouse 这类开源产品；

流处理分析后的结果，一方面作为存储持久化下来，另一方面也会进入告警系统，以主动发现问题来通知用户，如错误率超过指定阈值发出告警通知这样的需求等。

## 开源实现



### SkyWalking

#### 前置条件：

搭建skywalking需要三个镜像：

1.es 存储数据

2.oap 服务器

3.ui 界面

这里涉及到多个服务，采用docker-compose部署：

```dockerfile
version: '3.8'
services:
  elasticsearch:
    # image: docker.elastic.co/elasticsearch/elasticsearch-oss:${ES_VERSION}
    image: elasticsearch:7.9.0
    container_name: elasticsearch
    # --restart=always ： 开机启动，失败也会一直重启；
    # --restart=on-failure:10 ： 表示最多重启10次
    restart: always
    ports:
      - "9200:9200"
      - "9300:9300"
    healthcheck:
      test: [ "CMD-SHELL", "curl --silent --fail localhost:9200/_cluster/health || exit 1" ]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    environment:
      - discovery.type=single-node
      # 锁定物理内存地址，防止elasticsearch内存被交换出去,也就是避免es使用swap交换分区，频繁的交换，会导致IOPS变高；
      - bootstrap.memory_lock=true
      # 设置时区
      - TZ=Asia/Shanghai
      # - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1

  oap:
    image: apache/skywalking-oap-server:8.9.1
    container_name: oap
    restart: always
    # 设置依赖的容器
    depends_on:
      elasticsearch:
        condition: service_healthy
    # 关联ES的容器，通过容器名字来找到相应容器，解决IP变动问题
    links:
      - elasticsearch
    # 端口映射
    ports:
      - "11800:11800"
      - "12800:12800"
    # 监控检查
    healthcheck:
      test: [ "CMD-SHELL", "/skywalking/bin/swctl ch" ]
      # 每间隔30秒执行一次
      interval: 30s
      # 健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败；
      timeout: 10s
      # 当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
      retries: 3
      # 应用的启动的初始化时间，在启动过程中的健康检查失效不会计入，默认 0 秒。
      start_period: 10s
    environment:
      # 指定存储方式
      SW_STORAGE: elasticsearch
      # 指定存储的地址
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      SW_HEALTH_CHECKER: default
      SW_TELEMETRY: prometheus
      TZ: Asia/Shanghai
      # JAVA_OPTS: "-Xms2048m -Xmx2048m"

  ui:
    image: apache/skywalking-ui:8.9.1
    container_name: skywalking-ui
    restart: always
    depends_on:
      oap:
        condition: service_healthy
    links:
      - oap
    ports:
      - "8088:8080"
    environment:
      SW_OAP_ADDRESS: http://oap:12800
      TZ: Asia/Shanghai

```

执行成功后，docker会启动对应的容器

验证步骤：

打开localhost:8088 看下ui界面是否能访问

访问：[http://localhost:9200/](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A9200%2F) 是否有数据返回，以验证es是否正常启动

#### 部署

1.下载对应系统的skywalking的[agent](https://github.com/apache/skywalking-go/releases/tag/v0.4.0)

2.执行如下命令：

```shell
cd /path/to/skywalking && make build
```

3.执行成功会在bin文件夹下多出对应系统的可执行文件

不同的操作系统对应的**可执行文件**不同。例如，mac系统需选择skywalking-go-agent--darwin-amd64

4.打开Go项目，在main package中导入skywalking module

```
package main

import (
	_ "github.com/apache/skywalking-go"
)
```

5.配置config.yaml文件

配置文件示例：

```yaml
agent:
  # Service name is showed in UI.
  service_name: ${SW_AGENT_NAME:Your_ApplicationName}
  # To obtain the environment variable key for the instance name, if it cannot be obtained, an instance name will be automatically generated.
  instance_env_name: SW_AGENT_INSTANCE_NAME
  # Sampling rate of tracing data, which is a floating-point value that must be between 0 and 1.
  sampler: ${SW_AGENT_SAMPLE:1}
  meter:
    # The interval of collecting metrics, in seconds.
    collect_interval: ${SW_AGENT_METER_COLLECT_INTERVAL:20}

sampling:
  rate: 100

reporter:
  grpc:
    # The gRPC server address of the backend service.
    backend_service: ${SW_AGENT_REPORTER_GRPC_BACKEND_SERVICE:127.0.0.1:11800}
    # The maximum count of segment for reporting tracing data.
    max_send_queue: ${SW_AGENT_REPORTER_GRPC_MAX_SEND_QUEUE:5000}
    # The interval(s) of checking service and backend service
    check_interval: ${SW_AGENT_REPORTER_GRPC_CHECK_INTERVAL:20}
    # The authentication string for communicate with backend.
    authentication: ${SW_AGENT_REPORTER_GRPC_AUTHENTICATION:}
    # The interval(s) of fetching dynamic configuration from backend.
    cds_fetch_interval: ${SW_AGENT_REPORTER_GRPC_CDS_FETCH_INTERVAL:20}
    tls:
      # Whether to enable TLS with backend.
      enable: ${SW_AGENT_REPORTER_GRPC_TLS_ENABLE:false}
      # The file path of ca.crt. The config only works when opening the TLS switch.
      ca_path: ${SW_AGENT_REPORTER_GRPC_TLS_CA_PATH:}
      # The file path of client.pem. The config only works when mTLS.
      client_key_path: ${SW_AGENT_REPORTER_GRPC_TLS_CLIENT_KEY_PATH:}
      # The file path of client.crt. The config only works when mTLS.
      client_cert_chain_path: ${SW_AGENT_REPORTER_GRPC_TLS_CLIENT_CERT_CHAIN_PATH:}
      # Controls whether a client verifies the server's certificate chain and host name.
      insecure_skip_verify: ${SW_AGENT_REPORTER_GRPC_TLS_INSECURE_SKIP_VERIFY:false}

log:
  # The type determines which logging type is currently used by the system.
  # The Go agent wourld use this log type to generate custom logs. It supports: "auto", "logrus", or "zap".
  # auto: Automatically identifies the source of the log.
  #       If logrus is present in the project, it wourld automatically use logrus.
  #       If zap has been initialized in the project, it would use the zap framework.
  #       By default, it would use std errors to output log content.
  # logrus: Specifies that the Agent should use the logrus framework.
  # zap: Specifies that the Agent should use the zap framework.
  # The system must have already been initialized through methods such as "zap.New", "zap.NewProduction", etc.
  type: ${SW_AGENT_LOG_TYPE:auto}
  tracing:
    # Whether to automatically integrate Tracing information into the logs.
    enable: ${SW_AGENT_LOG_TRACING_ENABLE:true}
    # If tracing information is enabled, the tracing information would be stored in the current Key in each log.
    key: ${SW_AGENT_LOG_TRACING_KEY:SW_CTX}
  reporter:
    # Whether to upload logs to the backend.
    enable: ${SW_AGENT_LOG_REPORTER_ENABLE:true}
    # The fields name list that needs to added to the label of the log.(multiple split by ",")
    label_keys: ${SW_AGENT_LOG_REPORTER_LABEL_KEYS:}

plugin:
  # List the names of excluded plugins, multiple plugin names should be splitted by ","
  # NOTE: This parameter only takes effect during the compilation phase.
  excluded: ${SW_AGENT_PLUGIN_EXCLUDES:}
  config:
    http:
      # Collect the parameters of the HTTP request on the server side
      server_collect_parameters: ${SW_AGENT_PLUGIN_CONFIG_HTTP_SERVER_COLLECT_PARAMETERS:true}
    mongo:
      # Collect the statement of the MongoDB request
      collect_statement: ${SW_AGENT_PLUGIN_CONFIG_MONGO_COLLECT_STATEMENT:false}
    sql:
      # Collect the parameter of the SQL request
      collect_parameter: ${SW_AGENT_PLUGIN_CONFIG_SQL_COLLECT_PARAMETER:true}
```

设置参数的方式：

方式一：使用配置文件配置

方式二：设置环境变量

6.构建项目

```shell
sudo go build -toolexec "path/to/skywalking-go-agent -config path/to/config.yaml"
```

7.启动项目，观察控制台是否有对应设置的service

![trace](/images/trace.png)

##### 备注：

配置文件中的plugin是用于配置链路追踪所需要用到的插件。插件支持则直接能扫描到对应的依赖。如果插件不支持则需要手动添加span进行分析



#### 手动添加span

参考如下链接

https://skywalking.apache.org/zh/2023-10-18-skywalking-toolkit-trace/







### Ref:

https://www.alibabacloud.com/help/zh/arms/tracing-analysis/untitled-document-1690516495864?spm=a2c63.p38356.0.0.59bc6bd1ao2W05#6f7d899246n3f

https://github.com/alibabacloud-observability/skywalking-demo/tree/master/skywalking-go-demo/skywalking-go-agent-demo

https://skywalking.apache.org/zh/2023-06-01-quick-start-with-skywalking-go-agent/

https://juejin.cn/post/7250317954993619000
