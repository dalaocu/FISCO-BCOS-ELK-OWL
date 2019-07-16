

## 安装
### elasticsearch 

进入 elasticsearch的[下载页面](https://www.elastic.co/cn/downloads/elasticsearch)

我们以centos7.2系统为例


```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-linux-x86_64.tar.gz

tar xvfz elasticsearch-*.tar.gz
cd elasticsearch*
```

运行es：

```
 nohup bin/elasticsearch &
 
```

测试运行状况：

```
curl http://localhost:9200/

```

添加pipeline：

```
PUT _ingest/pipeline/bcos_pipeline
{
  "description" : "fisco-bcos2.0 pipeline",
  "version" : 1,
  "processors" : [
    {
      "grok" : {
        "field": "message",
        "patterns": ["%{LOGLEVEL:log_level}\\|%{TIMESTAMP_ISO8601:timestamp}\\|(\\[\\g\\:%{INT:group_id}\\])?(\\[%{DATA:module_name}\\])?(\\[%{DATA:sub_module_name}\\])?%{GREEDYDATA:content}"]
      }
    },
    {
      "grok" : {
        "field": "content",
        "patterns": ["([\\+\\s])*%{DATA:event_name},%{GREEDYDATA:event_info}"]
      }
    },
    {
      "grok" : {
        "field": "log.file.path",
        "patterns": ["\\S\/node%{INT:node_id}\\/log\\Slog"]
      }
    },
    {
        "kv": {
          "field": "event_info",
          "field_split":",",
          "value_split": "=",
          "target_field": "event_map"
        }
    }
  ],
  
  "on_failure" : [
    {
      "set" : {
        "field" : "error_msg",
        "value" : "{{ _ingest.on_failure_message }}"
      }
    }
  ]
}
```

获取pipeline:

```
GET _ingest/pipeline/bcos_pipeline
```

测试是否生效：

```
POST _ingest/pipeline/bcos_pipeline/_simulate
{
  "docs" : [
    { "_source": {"message": "info|2019-07-03 16:51:08.123659|[g:1][CONSENSUS][SEALER]++++++++++++++++ Generating seal on,blkNum=5,tx=0,nodeIdx=0,hash=c859f89f..."}}  
  ]
}
```

如果返回以下内容，代表配置正确：
```
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "error_msg" : "field [log] not present as part of path [log.file.path]",
          "log_level" : "info",
          "message" : "info|2019-07-03 16:51:08.123659|[g:1][CONSENSUS][SEALER]++++++++++++++++ Generating seal on,blkNum=5,tx=0,nodeIdx=0,hash=c859f89f...",
          "content" : "++++++++++++++++ Generating seal on,blkNum=5,tx=0,nodeIdx=0,hash=c859f89f...",
          "event_info" : "blkNum=5,tx=0,nodeIdx=0,hash=c859f89f...",
          "group_id" : "1",
          "sub_module_name" : "SEALER",
          "event_name" : "Generating seal on",
          "module_name" : "CONSENSUS",
          "timestamp" : "2019-07-03 16:51:08.123659"
        },
        "_ingest" : {
          "timestamp" : "2019-07-15T06:51:34.343Z"
        }
      }
    }
  ]
}

```




### kibana

进入 kibana的[下载页面](https://www.elastic.co/cn/downloads/elasticsearch)

我们以centos7.2系统为例


```
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.2.0-linux-x86_64.tar.gz

tar xvfz kibana-*.tar.gz
cd kibana*
```

运行kibana：

```
 nohup bin/kibana -H 0.0.0.0 &
 
```

测试运行状况：

```
curl http://localhost:5601/

```

导入kibana配置：

打开Management->saved objects -> import  在弹框中选择： settings/fisco-bcos-elk-owl.ndjson 文件，执行导入


验证导入：

打开dashboards，搜索fisco，查看是否存在fisco-bcos-dashboard界面


### Beats: HeartBeat

进入 heartbeat[下载页面](https://www.elastic.co/cn/downloads/beats/heartbeat)

我们以centos7.2系统为例


```
wget https://artifacts.elastic.co/downloads/beats/heartbeat/heartbeat-7.2.0-linux-x86_64.tar.gz

tar xvfz heartbeat-*.tar.gz
cd heartbeat*
```


编辑 heartbeat.yml ：

```
vim heartbeat.yml

```

添加以下内容：
```
output.elasticsearch:
  hosts: ["localhost:9200"]
setup.kibana:
  host: "localhost:5601"

heartbeat.monitors:
- type: tcp
  schedule: '@every 5s' 
  hosts: ["localhost:20200"]
  mode: any
```
准备：
```
heartbeat setup
```

运行:

```
 bash heartbeat -e
 
 ```

### Filebeat

进入 Filebeat[下载页面](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html)

我们以centos7.2系统为例


```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.2.0-linux-x86_64.tar.gz
tar xzvf filebeat-7.2.0-linux-x86_64.tar.gz
cd filebeat*
```


编辑 filebeat.yml ：

```
vim filebeat.yml

```

配置文件：
```

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log

output.elasticsearch:
  hosts: ["localhost:9200"]
  pipeline: bcos_pipeline
  
```

运行:

```
sudo chown root filebeat.yml 
sudo ./filebeat -e
 
 ```

curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.2.0-linux-x86_64.tar.gz


### Beats: Metricbeat

进入 Metricbeat[下载页面](https://www.elastic.co/cn/downloads/beats/metricbeat)

我们以centos7.2系统为例


```
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.2.0-linux-x86_64.tar.gz

tar xvfz metricbeat-*.tar.gz
cd metricbeat*
```

运行metricbeat：

```
 nohup bin/metricbeat &
 
```

编辑 metricbeat.yml ：

```
output.elasticsearch:
  hosts: ["localhost:9200"]
setup.kibana:
  host: "localhost:5601"
```
module初始化：
```
./metricbeat modules enable system
./metricbeat setup
```

运行metricbeat:

```
./metricbeat -e
```



