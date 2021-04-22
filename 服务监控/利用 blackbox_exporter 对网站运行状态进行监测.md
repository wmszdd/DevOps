本次部署blackbox_exporter主要是为了监测公司的服务状态，因为公司的服务暴露了一个健康监测的页面，可以利用他来进行监测。我的监控环境是用Kube-Prometheus清单文件部署的，可能不太通用，但是大致的原理是一样的，灵活去修改配置文件即可。

# 安装blackbox_exporter

## 下载安装包

`[root@local ~]# wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.18.0/blackbox_exporter-0.18.0.linux-amd64.tar.gz`

## 将安装包解压

```bash
[root@local ~]# tar -xf blackbox_exporter-0.18.0.linux-amd64.tar.gz
```

## 进入解压目录并启动服务

```bash
[root@local ~]# cd blackbox_exporter-0.18.0.linux-amd64/
[root@local ~]#  nohup ./blackbox_exporter &
```

## 查看服务运行状态

```bash
[root@local ~]#  nohup ./blackbox_exporter &^C
[root@local ~]# ss -ntulp | grep 9115
tcp    LISTEN     0      128    [::]:9115               [::]:*                   users:(("blackbox_export",pid=3143,fd=3))
```

# 自定义Prometheus配置文件

## 写入一个文件prometheus-additional.yaml

```bash
vim prometheus-additional.yaml
```

```bash
- job_name: 'blackbox-http'
  metrics_path: /probe
  params:
    modelue: [http_2xx]
  static_configs:
  - targets:
    - https://12bit.cn  #需要监控的url
    - http://els-prod.yiyiny.com
    - http://wanjia-nfcp.yiyiny.com
    - http://pqlp.yiyiny.com
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 47.116.130.219:9115  #部署blackbox_exporter主机ip
```

## 把这个文件创建为一个secret对象

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

![demo](https://i.loli.net/2021/04/13/Ty6LvftIVlcEng3.png)



