---
title: Prometheus 和 Grafana
date: 2024-8-19 23:54
tags: 
  - 监控与可视化
  - Prometheus
  - Grafana
  - Docker
  - Linux
categories:
  - [DevOps, 监控与可视化]
---

>时间：2024-8-18  
>版本信息：
> - Prometheus  V2.53.2
> - Grafana     V11.1.4


## Prometheus 和 Grafana 介绍

Prometheus 和 Grafana 是监控领域中经常搭配使用的两个开源工具，它们共同构成了一个强大的监控解决方案。

### Prometheus
一个开源的监控系统和时间序列数据库，主要用于收集和存储各种指标数据。它具有强大的查询语言 PromQL，可以对时间序列数据进行灵活的查询和聚合。

Prometheus 主要分为两个部分：服务端（Prometheus Server）和数据收集端（ Exporter）

#### Prometheus Server（服务端）
Prometheus Server 负责从配置的目标（例如 Node Exporter）抓取数据，并存储这些数据。它执行数据采集、存储、查询和报警等任务。

##### 主要组件：
- 数据采集：通过 HTTP 请求抓取暴露的时间序列数据。
- 时间序列数据库：存储和管理时间序列数据。
- 查询语言：提供 PromQL（Prometheus Query Language）用于查询数据。
- 报警管理：集成 Alertmanager 用于处理和发送报警通知。

#### Exporter（数据收集端）

Prometheus官方提供了很多Exporter，用于采集不同类型的指标数据，如Node Exporter、MySQL Exporter、Kafka Exporter等。


##### Node Exporter

Node Exporter 是一种常见的 Prometheus 导出器，用于从 Linux 系统上收集系统级的指标数据（如 CPU 使用率、内存、磁盘 IO 等）。它将这些数据暴露为 Prometheus 可抓取的格式。  
它的主要功能是：  
- 数据收集：从操作系统的各种指标中获取数据。
- 数据暴露：通过 HTTP 将这些数据暴露给 Prometheus Server。


### Grafana
一个开源的分析和可视化平台，可以连接到多个数据源（包括 Prometheus），并以图形化的方式展示数据。Grafana 提供了丰富的图表类型、仪表盘和报警功能，使得监控数据更加直观易懂。

#### Grafana 的主要功能
- 创建仪表盘： 将多个图表、表格等元素组合成一个仪表盘，以展示系统的整体运行状况。
- 自定义图表： 支持多种图表类型，如折线图、柱状图、饼图、热力图等，可以对数据进行灵活的展示。
- 设置报警： 当指标数据超出设定的阈值时，触发报警，及时通知相关人员。
- 探索数据： 提供一个交互式的查询界面，方便用户探索和分析数据。


----


## 以Docker方式安装Prometheus和Grafana

> 安装在 DockerHost 192.168.31.77


### 创建目录和配置文件

#### 创建目录
> 如果不想映射这两个数据，也可不用配置    
> 映射数据目录只是为了删除容器之后数据不会丢失  

```
mkdir -p /opt/prometheus/data
mkdir -p /opt/grafana/data
```

#### 修改目录权限
```
sudo chown -R 65534:65534 /opt/prometheus/data
sudo chmod -R 775 /opt/prometheus/data
```
> 上面的 65534:65534 是 Docker 中常用的 nobody 用户和组的 UID 和 GID，确保 Prometheus 容器能够写入数据。


```
sudo chown -R 472:472 /opt/grafana/data
```
> Grafana 容器中的进程通常以 grafana 用户（UID 为 472）运行，因此，你需要确保该用户对 /var/lib/grafana 有读写权限。
> 用户ID可以通过如：`docker inspect grafana | grep User` 之类的命令查看

也可以简单粗暴的让所有用户都有读写权限，如：
```
sudo chmod -R 777 /opt/prometheus/data
sudo chmod -R 777 /opt/grafana/data
```


#### 创建配置文件

为 Prometheus 创建一个配置文件 prometheus.yml：  
vim /opt/prometheus/prometheus.yml  
```
global:
  scrape_interval: 10s  # 抓取数据的时间间隔

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['192.168.31.77:9100']  # 监控宿主机的 Node Exporter
```
> 这只是一个示例，先跑起来


### 拉取镜像
```
docker pull prom/prometheus
docker pull grafana/grafana
```

> 需要科学上网，或者配置国内的镜像地址



### 启动容器

#### 启动 Prometheus 容器
```
docker run -d \
  --name prometheus \
  --restart unless-stopped \
  -p 9090:9090 \
  -v /opt/prometheus/data:/prometheus \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

访问 http://192.168.31.77:9090 来查看 Prometheus 的 Web 界面

> 有防火墙的要注意开放端口  
> 


#### 启动 Grafana 容器

```
docker run -d \
    --name grafana \
    --restart unless-stopped \
    -p 3000:3000 \
    -v /opt/grafana/data:/var/lib/grafana \
    grafana/grafana
```

访问 http://192.168.31.77:3000  

默认用户名密码是：amdin/admin ，第一次登录会提示修改密码



> 有防火墙时需要开放对应端口



### 在目标机器上安装Node Exporter
> 安装在 192.168.31.77 这台docker宿主机上

首先，从 Prometheus 官方的 GitHub 仓库下载 Node Exporter 的最新版本。

```
mkdir -p /opt/prometheus/node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

```

解压缩后运行
```
tar -zxvf node_exporter-1.8.2.linux-amd64.tar.gz 

cd node_exporter-1.8.2.linux-amd64

nohup ./node_exporter &
```
Node Exporter 默认会在端口 9100 上启动，并暴露 /metrics 端点。如果一切正常，你可以在浏览器中访问 http://<目标机器的IP地址>:9100/metrics，查看暴露的系统指标。  
如：  
http://192.168.31.77:9100/metrics

此时应该能在 prometheus 的 status菜单的 targets 中看到端点已连接

![1723970385587.png](https://img.wangwen135.top:23456/image/2024/08/66c1b36a68c7a.png)


## 配置

### 配置 Grafana

#### 添加 Prometheus 数据源：
- 左侧菜单 Connections -> Data Sources
- 点击Add data source
- 选择Prometheus
- 给数据源起一个有意义的名称，比如“Prometheus”
- URL：输入Prometheus服务器的HTTP API地址，例如：http://192.168.31.77:9090


#### 创建仪表盘：
- 选择 “Dashboards” 菜单，选择“Create dashboard”，选择“Add visualization”
- 选择刚刚添加的 Prometheus 数据源

##### 配置面板：
- Metric：选择要查询的指标，例如node_cpu_seconds_total
- 标签：根据需要过滤数据，例如{instance="your_host"}。
![1723971325393.png](https://img.wangwen135.top:23456/image/2024/08/66c1b745f2fc5.png)

给面板命名并保存即可


### 导入仪表盘
给一个个监控指标来配置图标的方式相对的麻烦，需要相关的经验也非常费时间，我们可以通过导入配置的方式快速搭建出专业的监控界面，而无需从头开始配置每一个图表和面板。可以提高效率，复用性和专业性。

> 官方网站上有各种各样的模板可以使用  

如我们要监控linux主机的各种指标：

- 打开：https://grafana.com/grafana/dashboards
- 搜索：Node Exporter，点击第一个，如：https://grafana.com/grafana/dashboards/1860-node-exporter-full/
- 点击右边的复制模板ID （1860）
- 回到普罗米修斯中点击创建面板 --> `Import dashboards from files or grafana.com.`
- 在上面粘贴ID，点击Load，下一步选择数据源

![1724043381275.png](https://img.wangwen135.top:23456/image/2024/08/66c2d08f31690.png)



----




## 其他
### 将Node Exporter配置为系统服务

为了确保 Node Exporter 在系统重启后自动启动，你可以将其配置为系统服务。

- 创建一个新的系统服务文件 /etc/systemd/system/node_exporter.service：
```
sudo vim /etc/systemd/system/node_exporter.service
```
- 在文件中添加以下内容：
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=nobody
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure

[Install]
WantedBy=default.target
```
> 如果你将 Node Exporter 安装在其他目录下，比如 /usr/local/bin，请相应地调整 ExecStart 路径。  
> 这里使用软连接的方式

- 创建软链接
```
ln -s /opt/prometheus/node_exporter/node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/node_exporter
```

- 重新加载 systemd 配置以应用更改：
```
sudo systemctl daemon-reload
```

- 启动 Node Exporter 服务：
```
sudo systemctl start node_exporter
```

- 验证:
```
sudo systemctl status node_exporter
```
访问：http://192.168.31.85:9100/metrics

- 设置开机自启动：
```
sudo systemctl enable node_exporter
```

### 中文配置
#### 设置 Grafana 语言为中文

点击页面左上角的个人头像，然后选择“Profile”（偏好设置）  
在“Preferences”页面中，找到“Language”（语言）选项，设置为中文（简体）  

#### 使用中文监控面板
如：
- https://grafana.com/grafana/dashboards/12633-linux/
- https://grafana.com/grafana/dashboards/8919
- https://grafana.com/grafana/dashboards/16098

**Linux主机详情**
（12633-linux）效果如下：

![1724081514391.png](https://img.wangwen135.top:23456/image/2024/08/66c3656c0ebdd.png)
 



