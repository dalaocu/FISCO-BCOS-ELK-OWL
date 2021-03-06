# FISCO-BCOS-ELK-OWL

### 1. 介绍

随着『公众联盟链』的兴起，FISCO-BCOS社区的发展也越来越兴旺；随着越来越多的应用上线，对系统、服务和应用程序可观察性领域的需求和创新也越来越迫切。

我发起这个项目，旨在使用业界最成熟的ELK技术栈，提供一个可快速使用的数据采集、可视化和监控的脚手架，从而帮助用户处理所有系统、服务和应用程序健康相关的全部数据，无论是日志、指标、运行状态测量数据，还是痕迹，都统统搞定。

我们期待更多的社区用户参与进来，一起为社区的建设添砖加瓦。


### 2. 实现功能

- 支持自动收集和解析FISCO-BCOS的底层日志

![FISCO-BCOS界面](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/dashboard.png)

- 支持按条目展示日志，可定制展示的字段

![日志收集](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/dashboard-log-view.png)

- 支持对日志内容进行解析，可支持KQL及Lucene语法检索

![日志收集](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/log-view.png)

- 支持实时滚动的日志展示

![OS监控](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/log_roll.png)

- 支持收集运行操作系统的指标，并可视化展示

![日志收集](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/monitor-os.png)

- 支持应用端口心跳检测，并提供可视化展示界面

![服务心跳监控](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/heartbeat.png)

- 支持收集ELK运行数据，并提供可视化展示界面

![ELK监控](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/monitor-elk.png)



### 3. 设计

![ELK设计图](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/photos/design.png)


FileBeat: 用来收集FISCO-BCOS的节点日志；

HeartBeat: 用来收集应用服务运行心跳数据；

MetricBeat: 用来收集服务器运行数据；

ElasticSearch： 提供数据存储、检索和解析的服务。使用了内置的ingest pipeline来解析数据；

Kibana： 提供数据可视化服务，支持自定义仪表盘、图形和各类视图的配置。



### 4. 安装方法

详情请参考： [安装手册](https://github.com/dalaocu/FISCO-BCOS-ELK-OWL/blob/master/install.md)

