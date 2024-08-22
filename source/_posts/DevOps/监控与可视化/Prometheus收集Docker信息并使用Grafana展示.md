---
title: Prometheus收集Docker信息并使用Grafana展示
date: 2024-8-22 23:34
tags: 
  - 监控与可视化
  - Prometheus
  - Grafana
  - Docker
categories:
  - [DevOps, 监控与可视化]
---


Prometheus 收集 Docker 信息时，通常使用 cAdvisor（Container Advisor）作为收集器。cAdvisor 是一个开源的工具，专门用于收集容器运行时的资源使用情况和性能数据，支持 Docker 容器。它能够监控 CPU、内存、网络、磁盘 I/O 等多种指标，并将这些数据暴露为 Prometheus 可以抓取的指标。

官方地址：  
https://github.com/google/cadvisor


在CentOS上运行参考地址：  
https://github.com/google/cadvisor/blob/master/docs/running.md

### 运行 cAdvisor 容器

你可以通过 Docker 启动一个 cAdvisor 容器

命令如下：
```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/cgroup:/cgroup:ro \
  --publish=8080:8080 \
  --detach=true \
  --restart unless-stopped \
  --name=cadvisor \
  google/cadvisor:latest
```
在 Docker 中，--volume（或者简写为 -v）选项用于将宿主机的文件或目录挂载到容器中。这里的命令示例中使用了多个 --volume 选项，每个选项都将宿主机上的目录或文件挂载到容器内的某个位置。以下是这些挂载的含义：

如：`--volume=/:/rootfs:ro：`
- 宿主机路径：/ （根目录）
- 容器内路径：/rootfs
- 挂载模式：ro （只读）
- 含义：将宿主机的根文件系统以只读模式挂载到容器的 /rootfs 目录中。这样容器能够访问宿主机的所有文件系统结构，但不能进行任何修改。

其他目录：  
- /var/run： 通常包含一些运行时的进程和服务相关的文件，如 Docker 守护进程的 Unix socket，这样容器可以读取这些文件，但无法修改。 
- /sys： 文件系统是 Linux 中用于存储和访问系统设备的信息和状态的虚拟文件系统，通过只读方式挂载，可以让容器获取系统的状态信息。
- /var/lib/docker：宿主机的 Docker 数据目录（存放容器、镜像、网络等信息），cAdvisor 通过访问这个目录，可以获取有关 Docker 容器的信息。
- cgroup 是 Linux 内核提供的一种机制，用于对系统资源（CPU、内存、I/O 等）进行分组管理和限制，通过访问 cgroup 文件系统，cAdvisor 可以获取和监控容器的资源使用情况。


### 查看指标
启动 cAdvisor 后，它会在 http://<cAdvisor_host>:8080/metrics 路径下暴露所有收集的指标数据。Prometheus 将按照配置的抓取间隔自动抓取这些指标。

如：
- http://192.168.31.77:8080/metrics  
- http://192.168.31.77:8080/containers/  
- http://192.168.31.77:8080/containers/docker  

![1724169316705.png](https://img.wangwen135.top:23456/image/2024/08/66c4bc675d8ac.png)

### 在 Prometheus 中配置抓取

在 Prometheus 的配置文件 prometheus.yml 中，添加一个 scrape job 来抓取 cAdvisor 暴露的指标。比如：

```
scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['<cAdvisor_host>:8080']

```
重启容器：
```
docker restart prometheus
```
> prometheus 也是docker安装的

查看：
![1724169204562.png](https://img.wangwen135.top:23456/image/2024/08/66c4bbf738057.png)


---

### Grafana可视化配置

要在 Grafana 中展示 cAdvisor 收集的 Docker 容器信息，可以使用以仪表盘模板。这些模板已经预先配置好，可以直接导入并使用，从而快速开始监控 Docker 容器的资源使用情况。


#### 使用模板

在仪表盘中选择新建 -> 导入，填如复制的模板ID，选择数据源，保存

#### 搜索模板：  
https://grafana.com/grafana/dashboards/?search=Docker

通过关键词如 "Docker" 或 "cAdvisor" 进行搜索

#### 可用模板如：  

- cAdvisor Dashboard
  - ID: 19792
- Docker Dashboard for Prometheus 中文版
   - ID：11558
- Docker overview with Cadvisor + docker state exporter + node exporter    
  - ID：21154
- Docker and system monitoring
  - ID: 893
- Docker Swarm and Containers
  - ID：609
- docker container & OS node(node_exporter, cadvisor)
  - ID: 16314
  - 中文 docker 容器 cadvisor 和主机指标面板
  - 主机指标 ( 可绑定到主机，需要 node-exporter 的 instance 标签值与 cadvisor 的 instance 标签值相同 )


----

### 将 cadvisor 与 node_exporter 指标关联

有的面板可以同时监控 cadvisor 和 node_exporter 指标，如需将 cadvisor 与 node_exporter 指标关联，需要将 cadvisor 和 node_exporter 的 instance 标签设置为相同的值 ( 例如主机名、ip地址等 )

> cAdvisor 和 node_exporter 是常用的两个数据源。cAdvisor 用于监控 Docker 容器的资源使用情况，而 node_exporter 用于监控主机级别的系统资源（如 CPU、内存、磁盘等）。

为了在同一个 Grafana 面板中同时展示这两者的数据，并确保它们的监控数据能够正确关联起来，你需要确保这两个数据源的 instance 标签具有相同的值。通常这个值是主机名或 IP 地址。

#### 如何设置 instance 标签为相同值

你需要在 prometheus.yml 配置文件中使用 relabel_configs 来覆盖 instance 标签的默认值。  
找到配置 cAdvisor 和 docker宿主机Node Exporter 的抓取任务的部分，并添加 relabel_configs 来替换 instance 标签的。  
如：
```
   # 监控Docker宿主机主机的 Node Exporter
  - job_name: 'docker_node_exporter' 
    static_configs:
      - targets: ['192.168.31.77:9100']  
    relabel_configs:
      - target_label: instance
        replacement: 'DockerHost'

  # 监控Docker相关信息
  - job_name: 'cadvisor' 
    static_configs:
      - targets: ['192.168.31.77:8080']
    relabel_configs:
      - target_label: instance
        replacement: 'DockerHost'
```
> - target_label 指定要替换或设置的目标标签。在这个例子中，instance 标签就是我们要替换的目标标签。
> - replacement 定义了 target_label 标签的新值。在这个例子中，我们将 instance 标签的值设置为 'DockerHost'。

重启
```
docker restart prometheus
```

在 Prometheus 的 Status -> Targets 中查看

![1724340279966.png](https://img.wangwen135.top:23456/image/2024/08/66c7583c44c01.png)

这样 cAdvisor 的 instance 标签将与 宿主机的 node_exporter 的 instance 标签一致。

展示效果如：
![1724340790569.png](https://img.wangwen135.top:23456/image/2024/08/66c75a3b3b3a4.png)

