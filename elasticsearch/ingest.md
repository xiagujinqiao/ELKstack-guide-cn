# Ingest 节点

Ingest 节点是 Elasticsearch 5.0 新增的节点类型和功能。其开启方式为：在 `elasticsearch.yml` 中定义：

```
node.ingest: false
```

Ingest 节点的基础原理，是：节点接收到数据之后，根据请求参数中指定的管道流 id，找到对应的已注册管道流，对数据进行处理，然后将处理过后的数据，按照 Elasticsearch 标准的 indexing 流程继续运行。

## 创建管道流

```
curl -XPUT http://localhost:9200/_ingest/pipeline/my-pipeline-id -d '
{
    "description" : "describe pipeline",
    "processors" : [
        {
            "convert" : {
                "field": "foo",
                "type": "integer"
            }
        }
    ]
}'
```

然后发送端带着这个 `my-pipeline-id` 发请求就好了。示例见本书 beats 章节的介绍。

## 测试管道流

想知道自己的 ingest 配置是否正确，可以通过仿真接口测试验证一下：

```
curl -XPUT http://localhost:9200/_ingest/pipeline/_simulate -d '
{
    "pipeline" : {
        "description" : "describe pipeline",
        "processors" : [
            {
                "set" : {
                    "field": "foo",
                    "value": "bar"
                }
            }
        ]
    },
    "docs" : [
        {
            "_index": "index",
            "_type": "type",
            "_id": "id",
            "_source": {
                "foo" : "bar"
            }
        }
    ]
}'
```

## 处理器

Ingest 节点的处理器，相当于 Logstash 的 filter 插件。事实上其主要处理器就是直接移植了 Logstash 的 filter 代码成 Java 版本。目前最重要的几个处理器分别是：

### convert

```
{
    "convert": {
        "field" : "foo",
        "type": "integer"
    }
}
```

### grok

```
    {
        "grok": {
            "field": "message",
            "patterns": ["my %{FAVORITE_DOG:dog} is colored %{RGB:color}"]
            "pattern_definitions" : {
                "FAVORITE_DOG" : "beagle",
                "RGB" : "RED|GREEN|BLUE"
            }
        }
    }
```

### gsub

```
{
    "gsub": {
        "field": "field1",
        "pattern": "\.",
        "replacement": "-"
    }
}
```

### date

```
    {
        "date" : {
            "field" : "initial_date",
            "target_field" : "timestamp",
            "formats" : ["dd/MM/yyyy hh:mm:ss"],
            "timezone" : "Europe/Amsterdam"
        }
    }
```
