# 为什么选择Prometheus

* Prometheus 使用 Golang 实现，运行速度更快；
* Prometheus 属于一站式监控告警平台，依赖少，功能齐全；
* Prometheus 支持对云的或容器的监控；
* Prometheus 数据查询语句表现力更强大，内置更强大的统计函数。

列举一些常见的 Google 软件基础架构：

* Borg：分布式任务管理系统，
* Borgmon：强大的监控报警系统；
* BigTable：分布式Key/Value存储系统；
* Google File System：分布式文件系统；
* ... ...

众所周知 Kubernetes 由 Google 分布式管理系统 Borg 演化而来，Prometheus 是一款开源的业务监控和时序数据库，可以看作是 Google 内部监控系统 Borgmon 的一个（非官方）实现。自然，Prometheus 对 Kubernetes 监控的支持更良好。