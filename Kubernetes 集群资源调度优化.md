**引言**

在[分布式系统](https://index-ayx-app.com.cn)中，日志不再只是排障的辅助手段，而是可观测性体系的核心数据源。随着服务实例数量增长，传统单机日志方案会暴露出采集分散、检索低效、关联困难等问题。基于 ELK（Elasticsearch、Logstash、Kibana）搭建分布式日志监控系统，能够实现日志统一采集、结构化检索、实时分析与可视化告警，是中大型后端架构的基础设施之一。

**核心原理分析**

ELK 的关键在于分工明确：Filebeat 或 Logstash 负责采集与传输，Logstash 进行清洗、字段解析和格式转换，Elasticsearch 提供高可用索引与全文检索能力，Kibana 则用于检索、聚合分析和仪表盘展示。实践中，日志必须尽量结构化，例如采用 JSON 格式输出，便于按 traceId、serviceName、level 等字段快速定位问题。对于高并发场景，还应引入消息队列缓冲写入压力，避免日志峰值拖垮业务链路。

**代码示例**

下面的 Logstash 配置可以直接解决“多行异常堆栈无法正确聚合”的常见问题，将 Java 异常按完整事件采集：

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [log_type] == "java" {
    multiline {
      pattern => "^\d{4}-\d{2}-\d{2}"
      negate => true
      what => "previous"
    }
    json {
      source => "message"
    }
    mutate {
      rename => { "host" => "source_host" }
      remove_field => ["message", "@version"]
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    index => "app-log-%{+YYYY.MM.dd}"
  }
}
```

该配置的价值在于：将异常堆栈合并为单条日志，避免检索时上下文丢失；同时将原始字符串解析为结构化字段，便于 Kibana 中按服务、环境、级别进行筛选和聚合。

**总结**

ELK 并不是简单的日志收集工具组合，而是一套面向分布式可观测性的工程化方案。真正可用的日志平台，应同时满足统一采集、结构化存储、低延迟检索和可视化分析四个目标。对于生产环境，建议结合采样策略、索引生命周期管理和告警规则设计，才能在性能、成本与可维护性之间取得平衡。

**相关技术资源**

- https://index-ayx-app.com.cn
