---
ayout: mypost
title: Prometheus部署操作
categories: [Prometheus]
extMath: true
---

# Prometheus部署操作

## 操作环境

Ubuntu22.4：192.168.0.104

## 下载二进制源码包

[官方下载地址]:https://prometheus.io/download/

此处需要注意的是要选择对应的操作系统以及芯片架构类型，我这里是用的树莓派刷的ubuntu系统，所以选择的是arm架构

node_exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-arm64.tar.gz
```

prometheus

```bash
apt install prometheus
# 选择上面或下面的都可以
wget https://github.com/prometheus/prometheus/releases/download/v2.37.5/prometheus-2.37.5.linux-arm64.tar.gz
```

## 部署

直接解压

```bash
tar  -zxvf node_exporter-1.5.0.linux-arm64.tar.gz
tar  -zxvf prometheus-2.37.5.linux-arm64.tar.gz
```

### 设置启动

```bash
cp -R node_exporter-1.2.0.linux-amd64 /usr/local/node_exporter
cd /usr/local/node_exporter
touch /usr/lib/systemd/system/node_exporter.service 
vim /usr/lib/systemd/system/node_exporter.service
```

加入以下代码：

```bash
[Unit]
Description=node_exporter
After=network.target
[Service]
Type=simple
User=root
ExecStart=/usr/local/node_exporter/node_exporter
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

node_exporter 服务并设置开机启动

```bash
systemctl daemon-reload
systemctl enable node_exporter.service
systemctl start node_exporter.service
systemctl status node_exporter.service
```

这时候可以打开本地网页进行连接测试：http://192.168.0.104:9100/metrics

由于Prometheus是go语言写的，所以不需要编译，安装的过程非常简单，仅需要解压然后运行

```bash
cp -R prometheus-2.28.1.linux-amd64 /usr/local/prometheuscd
cd /usr/local/prometheuscd 
vim prometheus.yml
# 最后增加如下配置，其中需要注意格式要求
- job_name: "nodes"
    static_configs:
    - targets: ["localhost:9100"]
```

设置prometheus系统服务,并配置开机启动

```bash
touch /usr/lib/systemd/system/prometheus.service
vim /usr/lib/systemd/system/prometheus.service
```

写入如下配置

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/prometheuscd/prometheus --config.file=/usr/local/prometheuscd/prometheus.yml --web.enable-lifecycle --storage.tsdb.path=/usr/local/prometheuscd/data --storage.tsdb.retention=60d
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Prometheus启动参数说明

- –config.file：prometheus的配置文件路径
- –web.enable-lifecycle：prometheus配置更改后可以进行热加载
- –storage.tsdb.path：监控数据存储路径
- –storage.tsdb.retention：数据保留时间

设置开机启动

```
systemctl daemon-reload
systemctl enable prometheus.service
systemctl start prometheus.service
systemctl status prometheus.service
```

热加载方式

```bash
curl -X POST  http://localhost:9090/-/reload
```

## Grafana安装

[官方文档]:https://grafana.com/docs/grafana/next/setup-grafana/installation/debian/

按照官方文档装完之后启动地址：http://192.168.0.104:3000

账号/密码：admin

第一次登录提示修改，我直接不修改了，只有我一个人用

登陆后如图

<img src="https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130128390.png" alt="image-20230213012839330"/>

选择先配置数据源

![image-20230213012927190](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130129243.png)

设置名字，主要就是红框里面的两个，地址要正确

![image-20230213013027021](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130130063.png)

### 配置Grafana图形显示Linux硬件信息

Grafana官方提供模板地址： https://grafana.com/grafana/dashboards

本次要导入的模板： https://grafana.com/grafana/dashboards/16098

![image-20230213013209319](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130132371.png)

填写好了之后点击Load，一路确定就可以了

![image-20230213013249861](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202302130132898.png)

```
systemctl daemon-reload
systemctl enable grafana-server.service
systemctl start grafana-server.service
systemctl status grafana-server.service
```

