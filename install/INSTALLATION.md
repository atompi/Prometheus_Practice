# Prometheus Stack 部署方案

## Prometheus Stack 简介

_PS: 为了后面行文方便，我把 Prometheus 有关组件统一命名为：Prometheus Stack，这些组件包括：Prometheus、Grafana、Alertmanager、\*\_exporter（众多数据收集器）等。_

Prometheus 是由 SoundCloud 开源监控告警解决方案，从 2012 年开始编写代码，再到 2015 年 github 上开源以来，已经吸引了 9k+ 关注，以及很多大公司的使用；2016 年 Prometheus 成为继 k8s 后，第二名 CNCF(Cloud Native Computing Foundation) 成员。

作为新一代开源解决方案，很多理念与 Google SRE 运维之道不谋而合。

### 它有什么特点？

+ 自定义多维数据模型(时序列数据由metric名和一组key/value标签组成)
+ 非常高效的存储 平均一个采样数据占 ~3.5 bytes左右，320万的时间序列，每30秒采样，保持60天，消耗磁盘大概228G。
+ 在多维度上灵活且强大的查询语言(PromQl)
+ 不依赖分布式存储，支持单主节点工作
+ 通过基于HTTP的pull方式采集时序数据
+ 可以通过push gateway进行时序列数据推送(pushing)
+ 可以通过服务发现或者静态配置去获取要采集的目标服务器
+ 多种可视化图表及仪表盘支持

### 关键词：时间序列数据

Prometheus 所有的存储都是按时间序列去实现的，相同的 metrics(指标名称) 和 label(一个或多个标签) 组成一条时间序列，不同的label表示不同的时间序列。

每条时间序列是由唯一的 指标名称 和 一组 标签 （key=value）的形式组成。

Prometheus的数据格式是简单的文本格式，可以直接阅读。其中，#号开头的是注释，除此之外，每一行一个数据项，数据名在前，值在后。{}中是标签，一条数据可以有多个标签。

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
http_request_count{endpoint="/a"} 10
http_request_count{endpoint="/b"} 200
http_request_count(endpoint="/c") 3
copy
```

+ 指标名称

一般是给监测对像起一名字，例如 http_requests_total 这样，它有一些命名规则，可以包字母数字之类的的。通常是以应用名称开头监测对像数值类型单位这样。例如：

```
- push_total
- userlogin_mysql_duration_seconds
- app_memory_usage_bytes
```

+ 标签

就是对一条时间序列不同维度的识别了，例如 一个http请求用的是POST还是GET，它的endpoint是什么，这时候就要用标签去标记了。最终形成的标识便是这样了

```
http_requests_total{method="POST",endpoint="/api/tracks"}
```

如果以传统数据库的理解来看这条语句，则可以考虑 http_requests_total是表名，标签是字段，而timestamp是主键，还有一个float64字段是值了。（Prometheus里面所有值都是按float64存储）

### 关键词：push vs pull model

![push_vs_pull](./imgs/push_vs_pull.png)

我们目前比较熟悉的监控系统系统，基本上都是第一种 push类型的，监控系统被动接受来自agent主动上报的各项健康指标数据，典型的监控系统是open-falcon；还有一种就是基于pull模型的，被监控系统向外暴露系统指标，监控系统主动去通过某些方式（通常是http）拉取到这些指标，最典型的是 prometheus；当然，同时支持pull类型和push类型的监控系统也是存在的，那就是经典的zabbix。

### 核心组件

+ Prometheus Server， 主要用于抓取数据和存储时序数据，另外还提供查询和 Alert Rule 配置管理。
+ client libraries，用于对接 Prometheus Server, 可以查询和上报数据的客户端类库，如client_python。
+ push gateway ，用于批量，短期的监控数据的汇总节点，主要用于业务数据汇报等。
+ 各种汇报数据的 exporters ，例如汇报机器数据的 node_exporter, 汇报 MongoDB 信息的 MongoDB exporter 等等。
+ 用于告警通知管理的 alertmanager 。

### 基础架构

官方的架构图如下：

![官方架构图](./imgs/prometheus_arch.svg)

大致使用逻辑是这样：
1. Prometheus server 定期从静态配置的 targets 或者服务发现的 targets 拉取数据。
2. 当新拉取的数据大于配置内存缓存区的时候，Prometheus 会将数据持久化到磁盘（如果使用 remote storage 将持久化到云端）。
3. Prometheus 可以配置 rules，然后定时查询数据，当条件触发的时候，会将 alert 推送到配置的 Alertmanager。
4. Alertmanager 收到警告的时候，可以根据配置，聚合，去重，降噪，最后发送警告。

## 现有监控项与替换方案：

|现有监控|监控方法|替换方案|
|:---:|:---|:---|
|Linux Server info|zabbix_agentd|可由node_exporter替代|
|nginx status|nginx_status.sh|可由hnlq715/nginx-vts-exporter代替
|sidekiq status|checksidekiqstatus.sh|可由client_python代替|
|gitsrv status|ps -ef \| grep ...|可由client_python代替|
|mysql|checkmysqlperformance.sh|可由mysqld_exporter代替|
|redis|redis-status.sh|可由redis_exporter替代|

## 告警方案

alertmanager + 163邮箱

告警规则

## 开始部署

部署分为：

+ go environment
+ Prometheus Server
+ alertmanager
+ Grafana
+ node_exporter
+ mysqld_exporter
+ redis_exporter
+ mission802/nginx_exporter
+ client_python

### go environment (all hosts)

1. 获取二进制包`go1.8.3.linux-amd64.tar.gz`
2. 解压二进制包，并移动到`/usr/local/`目录下

```bash
tar -zxf go1.8.3.linux-amd64.tar.gz
mv go /usr/local/
```

3. 修改环境变量`/etc/profile`

```bash
export PATH=$PATH:/usr/local/go/bin
```

4. 退出终端，重新登录；执行`go`，如下则成功：

```bash
git@app3:~/PrometheusStack$ go
Go is a tool for managing Go source code.

Usage:

	go command [arguments]

The commands are:

	build       compile packages and dependencies
	clean       remove object files
	doc         show documentation for package or symbol
	env         print Go environment information
	fix         run go tool fix on packages
	fmt         run gofmt on package sources
	generate    generate Go files by processing source
	get         download and install packages and dependencies
	install     compile and install packages and dependencies
	list        list packages
	run         compile and run Go program
	test        test packages
	tool        run specified go tool
	version     print Go version
	vet         run go tool vet on packages

Use "go help [command]" for more information about a command.

Additional help topics:

	c           calling between Go and C
	buildmode   description of build modes
	filetype    file types
	gopath      GOPATH environment variable
	environment environment variables
	importpath  import path syntax
	packages    description of package lists
	testflag    description of testing flags
	testfunc    description of testing functions

Use "go help [topic]" for more information about that topic.

```

### Prometheus Server (Prometheus Server host)

1. 获取二进制包`prometheus-1.7.1.linux-amd64.tar.gz`
2. 解压并移动到安装目录

```bash
tar -zxf prometheus-1.7.1.linux-amd64.tar.gz
mv prometheus-1.7.1.linux-amd64 /usr/local/prometheus-server
```

3. 配置prometheus-server

默认配置文件：`/usr/local/prometheus-server/prometheus.yml`
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']
```

检查安装环境是否正常：

```bash
git@app3:/usr/local/prometheus-server$ /usr/local/prometheus-server/prometheus --version
prometheus, version 1.7.1 (branch: master, revision: 3afb3fffa3a29c3de865e1172fb740442e9d0133)
  build user:       root@0aa1b7fc430d
  build date:       20170612-11:44:05
  go version:       go1.8.3
```

启动`prometheus-server`

```bash
nohup /usr/local/prometheus-server/prometheus -config.file /usr/local/prometheus-server/prometheus.yml -alertmanager.url http://localhost:9093 > /usr/local/prometheus-server/prometheus.log &
```

### alertmanager

1. 获取二进制包`alertmanager-0.8.0.linux-amd64.tar.gz`
2. 解压并移动到安装目录

```bash
tar -zxf alertmanager-0.8.0.linux-amd64.tar.gz
mv alertmanager-0.8.0.linux-amd64 /usr/local/alertmanager
```

3. 配置alertmanager

默认配置文件：`/usr/local/alertmanager/simple.yml`

```
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'
  # The auth token for Hipchat.
  hipchat_auth_token: '1234556789'
  # Alternative host for Hipchat.
  hipchat_url: 'https://hipchat.foobar.org/'

# The directory from which notification templates are read.
templates:
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  # The labels by which incoming alerts are grouped together. For example,
  # multiple alerts coming in for cluster=A and alertname=LatencyHigh would
  # be batched into a single group.
  group_by: ['alertname', 'cluster', 'service']

  # When a new group of alerts is created by an incoming alert, wait at
  # least 'group_wait' to send the initial notification.
  # This way ensures that you get multiple alerts for the same group that start
  # firing shortly after another are batched together on the first
  # notification.
  group_wait: 30s

  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 5m

  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 3h

  # A default receiver
  receiver: team-X-mails

  # All the above attributes are inherited by all child routes and can
  # overwritten on each.

  # The child route trees.
  routes:
  # This routes performs a regular expression match on alert labels to
  # catch alerts that are related to a list of services.
  - match_re:
      service: ^(foo1|foo2|baz)$
    receiver: team-X-mails
    # The service has a sub-route for critical alerts, any alerts
    # that do not match, i.e. severity != critical, fall-back to the
    # parent node and are sent to 'team-X-mails'
    routes:
    - match:
        severity: critical
      receiver: team-X-pager
  - match:
      service: files
    receiver: team-Y-mails

    routes:
    - match:
        severity: critical
      receiver: team-Y-pager

  # This route handles all alerts coming from a database service. If there's
  # no team to handle it, it defaults to the DB team.
  - match:
      service: database
    receiver: team-DB-pager
    # Also group alerts by affected database.
    group_by: [alertname, cluster, database]
    routes:
    - match:
        owner: team-X
      receiver: team-X-pager
    - match:
        owner: team-Y
      receiver: team-Y-pager


# Inhibition rules allow to mute a set of alerts given that another alert is
# firing.
# We use this to mute any warning-level notifications if the same alert is
# already critical.
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  # Apply inhibition if the alertname is the same.
  equal: ['alertname', 'cluster', 'service']


receivers:
- name: 'team-X-mails'
  email_configs:
  - to: 'team-X+alerts@example.org'

- name: 'team-X-pager'
  email_configs:
  - to: 'team-X+alerts-critical@example.org'
  pagerduty_configs:
  - service_key: <team-X-key>

- name: 'team-Y-mails'
  email_configs:
  - to: 'team-Y+alerts@example.org'

- name: 'team-Y-pager'
  pagerduty_configs:
  - service_key: <team-Y-key>

- name: 'team-DB-pager'
  pagerduty_configs:
  - service_key: <team-DB-key>
- name: 'team-X-hipchat'
  hipchat_configs:
  - auth_token: <auth_token>
    room_id: 85
    message_format: html
    notify: true
```

### Grafana

1. 创建`/usr/local/services/grafana`目录

```
sudo mkdir -p /usr/local/services/grafana
```

2. 获取二进制包并解压到目录`/usr/local/services/grafana`下

```
sudo tar -zxf grafana-4.4.3.linux-x64.tar.gz -C /usr/local/services/grafana --strip-components=1
```

3. 编辑配置文件`/usr/local/services/grafana/conf/defaults.ini`，修改`dashboards.json`段落下两个参数的值：

```
[dashboards.json]
enabled = true
path = /var/lib/grafana/dashboards
```

安装仪表盘:

```
sudo mkdir -p /var/lib/grafana/dashboards
# https://github.com/percona/grafana-dashboards.git
sudo tar -zxf grafana-dashboards.tar.gz -C /var/lib/grafana/dashboards --strip-components=1
```

启动grafana-server

```
cd /usr/local/services/grafana/
sudo nohup /usr/local/services/grafana/bin/grafana-server --homepath /usr/local/services/grafana > /home/atompi/grafana.log &
```

### node_exporter

1. 获取二进制包 `node_exporter-0.14.0.linux-amd64.tar.gz`
2. 解压到`/usr/local`目录下

```
tar -zxf node_exporter-0.14.0.linux-amd64.tar.gz
sudo mv node_exporter-0.14.0.linux-amd64 /usr/local/node_exporter
```

3. 运行node_exporter

```
sudo nohup /usr/local/node_exporter/node_exporter > /usr/local/node_exporter/node_exporter.log &
```

### mysqld_exporter

1. 获取二进制包 `mysqld_exporter-0.10.0.linux-amd64.tar.gz`
2. 解压到`/usr/local`目录下

```
tar -zxf mysqld_exporter-0.10.0.linux-amd64.tar.gz
sudo mv mysqld_exporter-0.10.0.linux-amd64.tar.gz /usr/local/mysqld_exporter
```

3. 创建`promuser`用户并授权

```
mysql> CREATE USER promuser;
mysql> GRANT REPLICATION CLIENT, PROCESS ON *.* TO 'promuser'@'127.0.0.1' identified by '123456';
mysql> GRANT SELECT ON performance_schema.* TO 'promuser'@'127.0.0.1';
```

4. 创建`/usr/local/mysqld_exporter/.my.cnf`文件

```
[client]
user=promuser
password=123456
```

5. 运行mysqld_exporter

```
sudo nohup /usr/local/mysqld_exporter/mysqld_exporter -config.my-cnf="/usr/local/mysqld_exporter/.my.cnf" > /usr/local/mysqld_exporter/mysqld_exporter.log &
```

### redis_exporter

1. 获取二进制包 `redis_exporter-v0.12.2.linux-amd64.tar.gz`
2. 解压到`/usr/local`目录下

```
tar -zxf redis_exporter-v0.12.2.linux-amd64.tar.gz
sudo mkdir /usr/local/redis_exporter
sudo mv redis_exporter /usr/local/redis_exporter
```

3. 运行redis_exporter

```
sudo nohup /usr/local/redis_exporter/redis_exporter redis//192.168.1.243:6379 > /usr/local/redis_exporter/redis_exporter.log &
```

### mission802/nginx_exporter

1. 获取二进制包 `nginx_exporter-1.0.linux-amd64.tar.gz`
2. 解压到`/usr/local`目录下

```
tar -zxf nginx_exporter-1.0.linux-amd64.tar.gz
sudo mv nginx_exporter-1.0.linux-amd64 /usr/local/nginx_exporter
```

3. 运行nginx_exporter

```
nohup /usr/local/nginx_exporter/nginx_exporter -nginx.scrape_uri=http://192.168.1.243/nginx_status > /usr/local/nginx_exporter/nginx_exporter.log &
```

### atompi/prompyclients

1. 获取`prompyclients`可执行文件 `prompyclients-1.0.1.tar.gz`
2. 解压到`/usr/local`目录下

```
tar -zxf prompyclients-1.0.1.tar.gz
sudo mv prompyclients-1.0.1 /usr/local/prompyclients
```

3. 运行prompyclients

```
nohup /usr/local/prompyclients/firstclient.py > /usr/local/prompyclients/prompyclients.log &
```

### systemd脚本

#### /etc/systemd/system/prometheus.service

```
# /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/prometheus-server/prometheus \
        -config.file=/usr/local/prometheus-server/prometheus.yml \
        -storage.local.path=/usr/local/prometheus-server/data

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/grafana.service

```
# /etc/systemd/system/grafana.service
[Unit]
Description=Grafana Server
Documentation=http://docs.grafana.org
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/services/grafana/bin/grafana-server --homepath /usr/local/services/grafana

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/node_exporter.service

```
# /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter Server
Documentation=https://github.com/prometheus/node_exporter
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/mysqld_exporter.service

```
# /etc/systemd/system/mysqld_exporter.service
[Unit]
Description=Mysqld Exporter Server
Documentation=https://github.com/prometheus/mysqld_exporter
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/mysqld_exporter/mysqld_exporter \
        -config.my-cnf="/usr/local/mysqld_exporter/.my.cnf"

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/redis_exporter.service

```
# /etc/systemd/system/redis_exporter.service
[Unit]
Description=Redis Exporter Server
Documentation=https://github.com/oliver006/redis_exporter
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/redis_exporter/redis_exporter \
        redis//192.168.1.243:6379

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/nginx_exporter.service

```
# /etc/systemd/system/nginx_exporter.service
[Unit]
Description=Nginx Exporter Server
Documentation=https://github.com/mission802/nginx_exporter
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/nginx_exporter/nginx_exporter \
        -nginx.scrape_uri=http://192.168.1.243/nginx_status

[Install]
WantedBy=multi-user.target
```

#### /etc/systemd/system/client_python.service

```
# /etc/systemd/system/client_python.service
[Unit]
Description=Client Python Server
Documentation=http://gitee.com/atompi/prompyclients
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/client_python/first_client.py

[Install]
WantedBy=multi-user.target
```