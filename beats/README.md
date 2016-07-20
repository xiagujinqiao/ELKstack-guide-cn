# Beats 平台

Beats 平台是 Elastic.co 从 packetbeat 发展出来的数据收集器系统。beat 收集器可以直接写入 Elasticsearch，也可以传输给 Logstash。其中抽象出来的 libbeat，提供了统一的数据发送方法，输入配置解析，日志记录框架等功能。

也就是说，所有的 beat 工具，在配置上，除了 input 以外，在 output、filter、shipper、logging、run-options 上的配置规则都是完全一致的。

## filter

5.0 版本后，beats 新增了简单的 filter 功能，用来完成事件过滤和字段删减：

```
filters:
    - drop_event:
        regexp:
            message: "^DBG:"
    - drop_fields:
        contains:
            source: "test"
        fields: ["message"]
    - include_fields:
        fields: ["http.code", "http.host"]
        equals:
            http.code: 200
        range:
            gte:
                cpu.user_p: 0.5
            lt:
                cpu.user_p: 0.8
```

## output

目前 beat 可以发送数据给 Elasticsearch、Logstash、File 和 Console 四个目的地址。

### Elasticsearch

beats 发送到 Elasticsearch 也是走 HTTP 接口。示例配置段如下：

```
output:
    elasticsearch:
        hosts: ["http://localhost:9200", "https://onesslip:9200/path", "anotherip"]
        parameters: {pipeline: my_pipeline_id}                         # 仅用于 Elasticsearch 5.0 以后的 ingest 方式
        username: "user"
        password: "pwd"
        index: "topbeat"
        bulk_max_size: 20000
        flush_interval: 5
        tls:
            certificate_authorities: ["/etc/pki/root/ca.pem"]
            certificate: "/etc/pki/client/cert.pem"
            certificatekey: "/etc/pki/client/cert.key"
```

* `hosts` 中可以通过 URL 的不同形式，来表示 HTTP 还是 HTTPS，是否有添加代理层的 URL 路径等情况。
* `index` 表示写入 Elasticsearch 时索引的前缀，比如示例即表示索引名为 `topbeat-yyyy.MM.dd`

### Logstash

beat 写入 Logstash 时，会配合 Logstash-1.5 后新增的 metadata 特性。将 beat 名和 type 名记录在 metadata 里。所以对应的 Logstash 配置应该是这样：

```
input {
    beats {
        port => 5044
    }
}
output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
    }
}
```

beat 示例配置段如下:

```
output:
    logstash:
        hosts: ["localhost:5044", "localhost:5045"]
        worker: 2
        loadbalance: true
        index: topbeat
```

这里 `worker` 的含义，是 beat 连到每个 host 的线程数。在 `loadbalance` 开启的情况下，意味着有 4 个worker 轮训发送数据。

### File

```
output:
    file:
        path: "/tmp/topbeat"
        filename: topbeat
        rotate_every_kb: 1000
        number_of_files: 7
```
### Console

```
output:
    console:
        pretty: true
```

## shipper

shipper 部分是一些和网络拓扑相关的配置，就目前来说，大多数是 packetbeat 独有的。

```
shipper:
    name: "my-shipper"
    tags: ["my-service", "hardware", "test"]
    ignore_outgoing: true
    refresh_topology_freq: 10
    topology_expire: 15
    geoip:
        paths:
            - "/usr/share/GeoIP/GeoLiteCity.dat"
```
## logging

```
logging:
    level: warning
    to_files: true
    to_syslog: false
    files:
        path: /var/log/mybeat
        name: mybeat.log
        keepfiles: 7
```
## run options

```
runoptions:
    uid=501
    gid=501
```
