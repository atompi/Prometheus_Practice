## Alert

Alert 是 Prometheus 的报警规则，由 Prometheus 主动发送到 Alertmanager。

## Alertmanager

Alertmanager 接收来自 Prometheus 的警报，将它们聚合成组，删除重复告警，过滤，然后发送通知到电子邮件，Pagerduty，Slack等。

## Bridge

Bridge 是一个从客户端库中提取样本并将其展示给非 Prometheus 监控系统的组件。例如，Python，Go和Java客户端可以将度量标准导出到 Graphite。

## Client library

Client library （客户端）库是某种语言的库（例如Go，Java，Python，Ruby），可以很容易地直接使用代码，编写自定义收集器来从其他系统获取指标，并将指标公开给 Prometheus。

## Collector

collector 是 exporter 的一部分，代表一组指标。如果它是直接检测的一部分，则可能是单个指标，如果是从另一个系统提取指标，则可能是多个指标。

## Direct instrumentation

Direct instrumentation 是作为程序源代码的一部分内联添加的。

## Endpoint

Endpoint （终端），可以被收集时序数据的来源，通常对应于一个单独的进程。