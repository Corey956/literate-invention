---
ayout: mypost
title: Prometheus监控系统认识
categories: [Prometheus]
extMath: true
---

# 监控系统

## 监控系统一般由4个功能组件：

- 指标数据采集
- 置标数据存储
- 置标数据趋势分析及可视化
- 警告

## 监控体系

### 系统层监控

- 系统监控：cpu、load、memory、swap、diskio、processes……
- 网络监控：网络设备、工作负载、网络延迟、丢包率……

### 中间件及基础设施类系统监控

- 消息中间件：kafka、rocketMQ、Rabbitmq等
- web服务容器：nginx、tomcat、apache等
- 数据库及缓存系统：mysql、Psql、mogodb、ES、redis等。
- 数据库连接池：ShardingSpere等
- 存储系统：ceph等

### 应用层监控

用于检测应用程序代码的状态和性能

### 业务层监控

业务接口：登录数、注册、搜索量等

## 三个监控方法论

### 黄金指标

源于google的SRE一书。（适用于应用及服务监控）

- 延迟：服务请求所需要的时长，例如http请求的平均时长
- 流量：衡量服务的容量需求，例如每秒处理的http请求数量或者其他事务数量。
- 错误：请求失败的情况，例如网站返回错误代码
- 饱和度：衡量资源的使用情况，用于展示服务器或应用程序资源，例如cpu、内存、IO、网络的使用量

### Netflix的USE方法

主要用于分析系统性能问题 （适用于主机、服务器指标监控）

- 使用率，关注系统资源的使用情况，包括不限于cpu、内存、io、网络等，100%使用率通常是系统性能瓶颈的标志
- 饱和度
- 错误

### red方法

基于黄金指标，结合kubernetes容器实践，适合云原生应用和微服务架构应用

red方法主要关注以下3种关键指标：

- rate ：每秒接收到的请求数
- errors：每秒失败的请求数
- duration：每个请求所花费的时间

# Prometheus（普罗米修斯）

## 概述

[Prometheus](https://github.com/prometheus)是一个开源系统监控和警报工具包，最初由 [SoundCloud](https://soundcloud.com/)构建，Prometheus是将其指标收集并存储为时间序列数据，即指标信息与记录时的时间戳以及称为标签的可选键值对一起存储

**时序数据**是在一段时间内通过重复测量而获得的观察值的集合；将这些观测值绘制于图形之上，有一个数据轴和时间轴。我们服务器的指标数据、应用程序的性能检测数据、网络数据等也都是时序数据。基于http/https，从配置文件中指定的网络端点（客户端）上周期性获取指标数据

普罗米修斯的主要特点是：

- 具有由指标名称和键/值对标识的时间序列数据的多维[数据模型](https://prometheus.io/docs/concepts/data_model/)
- PromQL，一种[灵活的查询语言](https://prometheus.io/docs/prometheus/latest/querying/basics/) 来利用这个维度
- 不依赖分布式存储；单个服务器节点是自治的
- 时间序列收集通过 HTTP 上的拉模型进行
- 通过中间网关支持[推送时间序列](https://prometheus.io/docs/instrumenting/pushing/)
- 通过服务发现或静态配置发现目标
- 多种图形和仪表板支持模式

首先Prometheus是支持在三种类型上进行一个抓取数据指标

### 管理告警

主要是负责实现报警的功能，rometheus 的警报分为两部分。Prometheus 服务器中的警报规则将警报发送到 Alertmanager

Alertmanager管理这些警报，包括including silencing、inhibition、aggregation以及通过电子邮件、随叫随到的通知系统和聊天平台等方法发送通知，也可以通过web进行自定义告警处理方式

### 组成部分

Prometheus 生态系统由多个组件组成，其中许多组件是可选的：

- [Prometheus Server](https://github.com/prometheus/prometheus)

Prometheus是组件中的核心部分，用于是收集和存储时间序列数据，提供PromQL查询语句的支持，内部有自己的Express Browser UI，并且这个UI可以通过直接PromQL实现数据查询的可视化

- Exporters

将监控数据采集的端点通过HTTP服务的形式暴露给Prometheus Server，Prometheus Server通过访问该Exporter提供的Endpoint端点，即可以获取到需要采集的监控数据

- PushGateway

主要是实现接收由 Client push 过来的指标数据，在指定的时间间隔，由主程序来抓取。由于 Prometheus 数据采集基于 Pull 模型进行设计，因此在网络环境的配置上必须要让 Prometheus Server 能够直接与 Exporter 进行通信。当这种网络需求无法直接满足时，就可以利用 PushGateway 来进行中转。可以通过 PushGateway 将内部网络的监控数据主动 Push 到 Gateway 当中。而 Prometheus Server 则可以采用同样 Pull 的方式从 PushGateway 中获取到监控数据。

- Alertmanager

管理告警，主要是负责实现报警功能。在 Prometheus Server 中支持基于 PromQL 创建告警规则，如果满足PromQL定义的规则，则会产生一条告警，而告警的后续处理流程则由 AlertManager 进行管理。在 AlertManager 中我们可以与邮件，Slack 等等内置的通知方式进行集成，也可以通过 Webhook 自定义告警处理方式。AlertManager 即 Prometheus 体系中的告警处理中心

大多数 Prometheus 组件都是用[Go](https://golang.org/)编写的，这使得它们易于构建和部署为静态二进制文件

下面是组织架构图

![Prometheus architecture](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130036782.png)

### Prometheus 数据模型 

Prometheus 仅用于以“键值”(key)形式储存时序式的聚合数，不支持存储文本信息；

- 其中的“键”称为指标，它通常意味着cpu速率、内存使用率、负载等
- 同一指标可能会适配到多个目标，比如cpu使用率这个指标，我们需要对50台服务器设备进行使用。所以它使用“标签”（labels）作为元数据，从而为指标添加更多的信息描述
- 这些标签可以作为过滤器进行指标过滤及运算。

### 指标类型 （metric type）

Prometheus 使用4种方法来描述监视的指标

- Counter

计数器，用于保存计数型数据，如网站访问量等

- Gauge

仪表盘，用于存储有着起伏特征的指标数据，如空间空闲大小等。

- Histogram

直方图，在一段时间范围内对数据进行采样，并将其计入可配置的存储中，后续可通过制定区间筛选样本，也可以统计样本总数，最后一般将数据展示为直方图。

- Summary

Histogram的扩展类型，用于表示一段时间内的数据采样结果（通常是请求持续时间或响应大小等），但它直接存储了分位数（通过客户端计算，然后展示出来），而不是通过区间计算

### 作业（job）和实例（Instance）

- Instance

实例可以理解为就是个target，在网络客户度啊你，实际上在多核心的服务器上就是一个instance就是代表一个CPU的核心

- job

也就是任务理解，通常类似功能的Instance的集合可以作为是一个job，

### PromQL

内置的一个查询语句，可以让用户进行实时的数据u查询以及聚合操作

其中提供了四种查询向量，如下：

- **时向量**- 一组时间序列，每个时间序列包含一个样本，所有样本共享相同的时间戳
- **范围向量**- 一组时间序列，其中包含每个时间序列随时间变化的一系列数据点
- **标量**- 一个简单的数字浮点值
- **String** - 一个简单的字符串值；目前未使用

### Alerts

在抓取到异常值之后，Prometheus支持通过报警机制向用户进行及时反馈，以便于用户可以及时采取相对应的措施

Prometheus server 仅负责生成报警指示，具体的报警行为由另一个独立的应用程序AlertManager负责

**报警指示由Prometheus server 基于用户提供的“报警规则”周期性计算生成**

**AlertManager接收到Prometheus server发来的报警指示后，基于用户定义的报警路由（route）向接收人（receivers）发送报警信息**