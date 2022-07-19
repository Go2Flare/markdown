OpenTelemetry
OTEL 是 OpenTelemetry 的简称， 是 CNCF 的一个可观测性项目，旨在提供可观测性领域的标准化方案，解决观测数据的数据模型、采集、处理、导出等的标准化问题，提供与三方 vendor 无关的服务。
OpenTelemetry 是一组标准和工具的集合，旨在管理观测类数据，如 Traces、Metrics、Logs 等 (未来可能有新的观测类数据类型出现)。目前已经是业内的标准。

OTLP
OTLP（全称 OpenTelemetry Protocol ）是 OpenTelemetry 原生的遥测信号传递协议，虽然在 OpenTelemetry 的项目中组件支持了Zipkin v2或Jaeger Thrift的协议格式的实现，但是都是以第三方贡献库的形式提供的。只有 OTLP 是 OpenTelemetry 官方原生支持的格式。OTLP 的数据模型定义是基于 ProtoBuf 完成的，如果你需要实现一套可以收集 OTLP 遥测数据的后端服务，那就需要了解里面的内容，对应可以参考代码仓库：opentelemetry-proto（https://github.com/open-telemetry/opentelemetry-proto）

Opentelemetry-Collector
OpenTelemetry Collector （以下简称“otel-collector”）针对如何接收、处理和导出遥测数据提供了与供应商无关的实现。它消除了运行、操作和维护多个代理/收集器的需要，以支持将开源可观察性数据格式（例如 Jaeger、Prometheus 等）发送到一个或多个开源或商业后端。此外，收集器让最终用户可以控制他们的数据。收集器是默认位置检测库导出其遥测数据。![otel-collecotr 架构](https://img-blog.csdnimg.cn/5416841d1cfe4a85aa54dd55363603ee.jpeg)


Opentelemetry-Java
Opentelemetry基于java语言开发的sdk，支持将数据通过各种exporter 推送到不同的观测平台。

Opentelemetry-JS
Opentelemetry 推出的基于前端 js 的链路追踪。

架构![架构图](https://img-blog.csdnimg.cn/9a498b2184b94a6894a959861e6fa180.png)


架构说明
1、应用 server 和 client 将 metric 、trace 数据通过 otlp-exporter push 到 otel-collector

2、front-app 为前端链路，将链路信息 push 到 otel-collector，并访问应用服务 API

3、otel-collector 对数据进行收集、转换后，将数据 push 到 Jaeger、Zipkin

4、同时 Prometheus 从 otel-collector pull 数据。

日志两种推送方式：

方式一：通过 OTLP 上报日志

应用 server 和 client 将 log 通过 otlp-exporter push 到 otel-collector，再通过 otel-collector exporter 到 Elasticsearch。**由于 Opentelemetry log 方面还不稳定，所以推荐 log 单独处理，不走 otel-collector，在测试过程中也发下了 同时配置 log 和 metric 存在冲突问题，主要表现在 otel-collector 上，等待官方修复吧。**

方式二：通过 Logback-logstash 上报日志

应用 server 和client将 log 通过 Logback-logstash 推送到 logstash。

otel-collector 配置了四个 exporter.

```
  prometheus:
    endpoint: "0.0.0.0:8889"
    const_labels:
      label1: value1
  zipkin:
    endpoint: "http://otel_collector_zipkin:9411/api/v2/spans"
    format: proto
  jaeger:
    endpoint: otel_collector_jaeger:14250
    tls:
      insecure: true
  elasticsearch:
    endpoints: "http://192.168.0.17:9200"
# 注意，所有的应用都部署在同一个机器上，机器 ip 为 192.168.0.17。如果应用和一些中间件单独分开部署，则注意修改对应的 IP。如果是云服务器，则注意开放相关端口，以免访问失败。
```

安装部署
安装 Opentelemetry-Collector
源码地址
https://github.com/lrwh/observable-demo/tree/main/opentelemetry-collector-to-all

配置 otel-collector-config.yaml
新增 collecter 配置，配置1个recevier（otlp）、4个exporter(prometheus、zipkin、jaeger 和 elasticsearch。

```
receivers:
  otlp:
    protocols:
      grpc:
      http:
        cors:
          allowed_origins:
            - http://*
            - https://*
exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
    const_labels:
      label1: value1
  zipkin:
    endpoint: "http://otel_collector_zipkin:9411/api/v2/spans"
    format: proto

  jaeger:
    endpoint: otel_collector_jaeger:14250
    tls:
      insecure: true
  elasticsearch:
    endpoints: "http://192.168.0.17:9200"

processors:
  batch:

extensions:
  health_check:
  pprof:
    endpoint: :1888
  zpages:
    endpoint: :55679

service:
  extensions: [pprof, zpages, health_check]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [zipkin, jaeger]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [elasticsearch]
```


通过 docker-compose 安装 otel-collector

```
version: '3.3'

services:
    jaeger:
        image: jaegertracing/all-in-one:1.29
        container_name: otel_collector_jaeger
        ports:
            - 16686:16686
            - 14250
            - 14268
    zipkin:
        image: openzipkin/zipkin:latest
        container_name: otel_collector_zipkin
        ports:
            - 9411:9411
    # Collector
    otel-collector:
        image: otel/opentelemetry-collector:0.50.0
        command: ["--config=/etc/otel-collector-config.yaml"]
        volumes:
            - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
        ports:
            - "1888:1888"   # pprof extension
            - "8888:8888"   # Prometheus metrics exposed by the collector
            - "8889:8889"   # Prometheus exporter metrics
            - "13133:13133" # health_check extension
            - "4317:4317"        # OTLP gRPC receiver
            - "4318:4318"        # OTLP http receiver
            - "55670:55679" # zpages extension
        depends_on:
            - jaeger
            - zipkin
    prometheus:
        container_name: prometheus
        image: prom/prometheus:latest
        volumes:
            - ./prometheus.yaml:/etc/prometheus/prometheus.yml
        ports:
            - "9090:9090"
    grafana:
        container_name: grafana
        image: grafana/grafana
        ports:
            - "3000:3000"
```


配置 Prometheus

```
scrape_configs:

  - job_name: 'otel-collector'
    scrape_interval: 10s
    static_configs:
      - targets: ['otel-collector:8889']
      - targets: ['otel-collector:8888']
```


启动容器

```
docker-compose up -d
```


查看启动情况

```
docker-compose ps
```





Docker 安装 ELK
采用 Docker 安装 ELK ,简单又方便，相关组件版本为7.16.2。

拉取镜像

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.16.2
docker pull docker.elastic.co/logstash/logstash:7.16.2
docker pull docker.elastic.co/kibana/kibana:7.16.2
```


配置目录

```
# Linux 特有配置

sysctl -w vm.max_map_count=262144
sysctl -p
```



```
# Linux 配置结束

mkdir -p ~/elk/elasticsearch/plugins
mkdir -p ~/elk/elasticsearch/data
mkdir -p ~/elk/logstash
chmod 777 ~/elk/elasticsearch/data
```


Logstash配置

```
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "springboot-logstash-demo-%{+YYYY.MM.dd}"
  }
}
```


input 参数说明：

```
tcp : 为tcp协议。
port： tcp 端口
codec：json行解析
```

Docker-compose 配置

```
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.16.2
    container_name: elasticsearch
    volumes:
      - ~/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - ~/elk/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #设置使用jvm内存大小
      - "ingest.geoip.downloader.enabled=false" # (Dynamic, Boolean) If true, Elasticsearch automatically downloads and manages updates for GeoIP2 databases from the ingest.geoip.downloader.endpoint. If false, Elasticsearch does not download updates and deletes all downloaded databases. Defaults to true.
    ports:
      - 9200:9200
  logstash:
    image: docker.elastic.co/logstash/logstash:7.16.2
    container_name: logstash
    volumes:
      - ~/elk/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
  kibana:
    image: docker.elastic.co/kibana/kibana:7.16.2
    container_name: kibana
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    environment:
      - "elasticsearch.hosts=http://es:9200" #设置访问elasticsearch的地址
    ports:
      - 5601:5601
```

启动容器

```
docker-compose up -d
```

查看启动情况

```
docker-compose ps
```

![查看elk启动情况](https://img-blog.csdnimg.cn/ca13fa39f24c4f9488339e43d140ffa1.png)