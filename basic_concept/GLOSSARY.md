<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [名词解释](#名词解释)
	- [Alert](#alert)
	- [Alertmanager](#alertmanager)
	- [Bridge](#bridge)
	- [Client library](#client-library)
	- [Collector](#collector)
	- [Direct instrumentation](#direct-instrumentation)
	- [Endpoint](#endpoint)
	- [Exporter](#exporter)
	- [Instance](#instance)
	- [Job](#job)
	- [Notification](#notification)
	- [Promdash](#promdash)
	- [PromQL](#promql)
	- [Pushgateway](#pushgateway)
	- [Remote Read](#remote-read)
	- [Remote Read Endpoint](#remote-read-endpoint)
	- [Remote Write](#remote-write)
	- [Remote Write Adapter](#remote-write-adapter)
	- [Remote Write Endpoint](#remote-write-endpoint)
	- [Silence](#silence)
	- [Target](#target)

<!-- /TOC -->

# 名词解释

## Alert

Alert 是 Prometheus 的报警规则，由 Prometheus 主动发送到 Alertmanager。

## Alertmanager

[Alertmanager](https://prometheus.io/docs/alerting/overview/) 接收来自 Prometheus 的警报，将它们聚合成组，删除重复告警，屏蔽标记为静音（ silence ）的告警，然后发送通知到电子邮件，Pagerduty，Slack等。

## Bridge

Bridge 是一个从客户端库中提取样本并将其展示给非 Prometheus 监控系统的组件。例如，Python，Go和Java客户端可以将度量标准导出到 Graphite。

## Client library

Client library （客户端）库是某种语言的库（例如Go，Java，Python，Ruby），可以很容易地直接使用代码，编写自定义收集器来从其他系统获取 metrics ，并将 metrics 暴露给 Prometheus。

## Collector

collector 是 exporter 的一部分，代表一组 metrics 。如果它是直接检测的一部分，则可能是单个 metrics ，如果是从另一个系统提取 metrics ，则可能是多个 metrics 。

## Direct instrumentation

Direct instrumentation 是作为程序源代码的一部分内联添加的。

## Endpoint

Endpoint （终端），可以被收集时序数据的来源，通常对应于一个单独的进程。

## Exporter

Exporter 是一个二进制文件，通常通过将非 Prometheus 格式的 metrics 转换为 Prometheus 支持的格式来暴露给 Prometheus metrics 。

## Instance

Instance （实例）是唯一标识作业中的目标的标签。

## Job

Job （作业）是具有相同目的的一组目标（例如，监视一组为了可伸缩性或可靠性而复制的类似进程）被称为作业。

## Notification

Notification （通知）代表一组警报，由 Alertmanager 发送给电子邮件 ， Pagerduty ， Slack等。

## Promdash

Promdash 是 Prometheus 的本地仪表板生成器。它已被弃用，取而代之的是功能更强大的 [Grafana](https://prometheus.io/docs/visualization/grafana/) 。

## PromQL

[PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) 是 Prometheus 的查询语言。它允许多种操作，包括聚合，切片和切割，预测和连接。

## Pushgateway

[Pushgateway](https://prometheus.io/docs/instrumenting/pushing/) 持续不断的从批处理作业中推出的 metrics 拉取数据。 Prometheus 可以直接从 Pushgateway 获取数据，使得在 Prometheus 终止之后仍然可以继续收集这些作业的 metrics 。

## Remote Read

Remote Read （远程读取）是 Prometheus 的一个特性，允许 Prometheus 从其他系统（如长期存储）作为查询的一部分读取时间序列。

## Remote Read Endpoint

Remote Read Endpoint （远程读取节点）是 Prometheus 进行远程读取时所读取的对象。

## Remote Write

Remote Write （远程写入）是 Prometheus 的一个特性，可以将收集的监控样本随时发送到其他系统，如长期存储。

## Remote Write Adapter

Remote Write Adapter （远程写入适配器）并非所有系统都直接支持远程写入。 Prometheus 和另一个系统之间有一个远程写入适配器，将远程写入的样本转换成其他系统可以识别的格式。

## Remote Write Endpoint

Remote Write Endpoint （远程写入节点）是 Prometheus 在进行远程写入时所写入的对象。

## Silence

Silence （静音）Alertmanager 中的 silence 防止通知中包含被静音（屏蔽）的警报。

## Target

Target （目标）是一个对象的定义。例如，要应用哪些标签，连接所需的身份验证或定义收集的对象等信息。