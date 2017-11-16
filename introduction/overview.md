# 什么是 Prometheus

[Prometheus](https://prometheus.io/) 是由 `SoundCloud` 开源监控告警解决方案，从 2012 年开始编写代码，再到 2015 年 [github](https://github.com/prometheus/prometheus) 上开源以来，已经吸引了 12k+ 关注，以及很多大公司的使用；2016 年 Prometheus 成为继 k8s 后，第二名 CNCF([Cloud Native Computing Foundation](https://cncf.io/)) 成员。作为新一代开源解决方案，很多理念与 Google SRE 运维之道不谋而合。

# 它有什么特点

+ 多维数据模型以及由 metrics 名和一组 key/value 标签组成的时序数据
+ 在多维度上灵活且强大的查询语言([PromQl](https://prometheus.io/docs/querying))
+ 不依赖分布式存储，支持单主节点工作
+ 通过基于 HTTP 的 pull 方式采集时序数据
+ 支持通过 Pushgateway 进行时序数据跨网关推送([pushing time series](https://prometheus.io/docs/instrumenting/pushing/))
+ 支持通过服务发现或者静态配置去获取要采集的目标服务器
+ 支持多种可视化图表及仪表盘([Grafana](https://grafana.com/))

# 它由什么构成

+ Prometheus 主服务器，用来收集和存储时间序列数据
+ 用于应用程序埋点的 client 代码库
+ 用于短时 jobs 的 push gateway
+ 特殊用途的数据收集工具包 exporter（包括HAProxy、StatsD、Ganglia等)
+ 用于报警的 alertmanager
+ 命令行查询工具
+ Prometheus Dashboard 展现的绝佳工具 Grafana

大部分 Prometheus 组建都由 go 实现，这使它们易于构建和部署为静态的二进制文件。

# 架构图

此图说明 Prometheus 和它的一些生态系统组成部分的架构关系：

![Architecture](_images/prometheus_arch.svg)

Prometheus 直接或通过 Pushgateway 从各个 job 中爬取时序数据。它在本地存储所有的样品，对这些数据运行规则进行过滤、汇总并记录新的时序数据，同时可以从现有数据生成警报。Grafana或其它API可以将所采集的数据用于可视化展示。

# 它能做什么

Prometheus 可以很好地记录任何纯数字时间序列。它既适用于以服务器为中心的监控，也适用于高度动态的面向服务的体系结构的监控。在微服务的世界中，它支持多维数据收集和查询是一个独特的优势。

# 它不能做什么

Prometheus 的价值在于可靠性，即使在某些时间节点收集失败情况下，您也可以始终查看有关系统的统计信息。如果您需要100％的准确性（例如按请求计费），Prometheus并不是一个好的选择，因为收集的数据可能不够详细和完整。在这种情况下，您最好使用其他系统来收集和分析数据以进行计费，Prometheus 则可以进行其他监控。