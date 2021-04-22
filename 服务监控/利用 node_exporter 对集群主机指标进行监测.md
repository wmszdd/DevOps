# 部署node_exporter

**需要在监控的所有节点部署**

本次部署node_exporter主要是为了监测公司服务器运行状态，我的监控环境是用Kube-Prometheus清单文件部署的，可能不太通用，但是大致的原理是一样的，灵活去修改配置文件即可。

## 下载安装包

```bash
[root@work-node ~]# wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz^C
```

## 解压并启动服务

```bash
[root@work-node ~]# tar -xf node_exporter-1.1.2.linux-amd64.tar.gz
[root@work-node ~]# nohup ./node_exporter &
```

## 查看服务状态

```bash
[root@work-node ~]# ss -ntulp | grep :9100
tcp    LISTEN     0      128    [::]:9100               [::]:*                   users:(("node_exporter",pid=12987,fd=3))
```

# 自定义Prometheus配置文件

## 写入一个文件prometheus-additional.yaml

```bash
vim prometheus-additional.yaml
```

```bash
- job_name: node-exporter   #一个job下面可以设置多个targets
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - 192.168.1.10:9100
    - 47.116.130.219:9100
```

## 把这个文件新建为一个secret对象

**创建之前需要删除默认的**

```bash
[root@kube-node01 manifests]# kubectl delete secrets additional-configs -n monitoring
```

**新建建**

```bash
kubectl create secret generic additional-configs --from-file=prometheus-additional.yaml -n monitoring
secret "additional-configs" created
```

## 修改prometheus-prometheus.yaml

```bash
[root@kube-node01 manifests]# vim prometheus-prometheus.yaml
```

```bash

securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  additionalScrapeConfigs:  #新增
    name: additional-configs    #新增
    key: prometheus-additional.yaml #新增
  serviceAccountName: prometheus-k8s
  serviceMonitorNamespaceSelector: {}
```

## 添加完成后，apply下这个文件

```bash
[root@kube-node01 manifests]# kubectl apply -f prometheus-prometheus.yaml
prometheus.monitoring.coreos.com “k8s” configured
```

## 验证

访问`http://192.168.1.10:8002/targets`进行验证

效果图展示：

![node_exporter-demo.png](https://i.loli.net/2021/04/13/8ebMVuLqSKOEksN.png)