<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [基本概念](#基本概念)
	- [DATA MODEL （数据模型）](#data-model-数据模型)
		- [Metric names and labels （ metrics 名称和标签）](#metric-names-and-labels-metrics-名称和标签)
		- [Samples （样本）](#samples-样本)
		- [Notation （符号）](#notation-符号)
	- [METRIC TYPES （ metrics 数据类型）](#metric-types-metrics-数据类型)
		- [Counter （计数器）](#counter-计数器)
		- [Gauge](#gauge)
		- [Histogram](#histogram)

<!-- /TOC -->

# 基本概念

## DATA MODEL （数据模型）

Prometheus 从根本上将所有数据存储为[时间序列（ time series） ](http://en.wikipedia.org/wiki/Time_series)：属于同一个 metrics 的时间戳值的数据流和相同的一组标记维度。 除了存储的时间序列之外， Prometheus 还可以生成临时导出的时间序列作为查询的结果。

### Metric names and labels （ metrics 名称和标签）

每个时间序列都由其 metrics 名称和一组键值对（也称为标签）唯一标识。

metrics 名称指定被测量的系统的一般特征（例如， `http_requests_total` - 接收到的 HTTP 请求的总数）。它可以包含 ASCII 字母和数字，以及下划线和冒号，即必须匹配正则表达式 `[a-zA-Z_:][a-zA-Z0-9_:]*` 。

标签使 Prometheus 的维度数据模型成为可能：相同 metrics 名称的任何给定的标签组合标识该 metrics 的特定维度实例（如：所有使用 `POST` 方法到 `/api/tracks` 处理程序的 HTTP 请求）。查询语言允许基于这些维度进行筛选和聚合。更改任何标签值（包括添加或删除标签）将创建新的时间序列。

标签名称可以包含 ASCII 字母，数字以及下划线。它们必须匹配正则表达式 `[a-zA-Z_][a-zA-Z0-9_]*` 。以 `__` 开头的标签名称保留供内部使用。

标签值可能包含任何 Unicode 字符。

另请参阅[命名 metrics 和标签的最佳实践](https://prometheus.io/docs/practices/naming/)。

### Samples （样本）

样本形成实际的时间序列数据。每个样品包括：

    * 一个 float64 值
    * 一个毫秒精度的时间戳

### Notation （符号）

给定一个 metrics 名称和一组标签，时间序列通常用这个标记来标识：

```
<metric name>{<label name>=<label value>, ...}
```

例如，metrics name 为 `api_http_requests_total` ，标签 `method="POST"` ， `handler="/ messages"` 的时间序列可以这样写：

```
api_http_requests_total{method="POST", handler="/messages"}
```

这与 [OpenTSDB](http://opentsdb.net/) 使用的符号相同。

## METRIC TYPES （ metrics 数据类型）

Prometheus 客户端库提供了四种核心 metrics 数据类型。目前这些功能仅在客户端库和有线协议（ wire protocol ）中有所不同（为了使 API 适合特定类型的使用）。Prometheus 服务端尚未使用类型信息，而是将所有数据变为无类型的时间序列。这在未来端更新可能会有所改变。

### Counter （计数器）

计数器是一个累计类型端 metrics ，它代表了一个只有没次累计的数值。通常使用计数器来计数服务的请求、完成的任务、发生的错误等等。计数器不应该被用来显示其数目也可以减少的项目的当前计数，如：当前正在运行的进程的数量，这个用例适合使用 Gauges。

Counter 的客户端库使用文档：

* [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Counter)
* [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Counter.java)
* [Python](https://github.com/prometheus/client_python#counter)
* [Ruby](https://github.com/prometheus/client_ruby#counter)

### Gauge

Gauge 代表一个可以任意上下的单个数值。

Gauge 通常用于测量值，如温度或当前的内存使用情况，但也可以上下“计数”，如正在运行的进程的数量。

Gauge 的客户端库使用文档：

* [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Gauge)
* [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Gauge.java)
* [Python](https://github.com/prometheus/client_python#gauge)
* [Ruby](https://github.com/prometheus/client_ruby#gauge)

### Histogram

Histogram （直方图）对观察结果进行采样（通常是请求持续时间或响应大小），并将其计入可配置的 buckets 中。它也提供了所有观测值的总和。

metrics 名称为 `<basename>` 的直方图在收集期间公开多个时间序列：

* 观察桶的累计计数器显示为 `<basename>_bucket {le="<upper inclusive bound>"}`