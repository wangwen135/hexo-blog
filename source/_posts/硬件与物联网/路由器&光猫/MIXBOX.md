---
title: MIXBOX
date: 2022-06-05 23:41
tags: 
  - 路由器&光猫
categories:
  - [硬件与物联网, 路由器&光猫]
---

很好的工具，用了FTP、Aria2、FileBrowser、AliDDNS 等

### github 地址
https://github.com/monlor/MIXBOX-ARCHIVE


>下面为操作步骤记录


### 安装

#### 一键安装
```
sh -c "$(curl -kfsSl https://cdn.jsdelivr.net/gh/monlor/mbfiles/install.sh)" && source /etc/profile &> /dev/null

```

过程记录
```
***********************************************
**                                           **
**            Welcome to MIXBOX !            **
**                                           **
***********************************************
【Tools】: 请按任意键安装工具箱(Ctrl + C 退出).

请在以下路径中选择一个合适的工具箱安装位置和一个用户文件目录：
小米路由器硬盘版推荐 工具箱安装位置：/etc，用户目录：/userdisk/data
小米路由器普通版推荐 工具箱安装位置：/etc，用户目录：/extdisks/sda*
如果未插入u盘，用户目录可与工具箱安装位置相同！
1./
2./dev
3./tmp
4./
5./extdisks
6./data
7./userdisk
8./userdisk/data
9./etc
...
...
请输入你的工具箱安装路径[可手动输入路径]：9
请输入你的用户文件目录[可手动输入路径]：8
【Tools】: 下载工具箱文件...
【Tools】: 解压工具箱文件
【Tools】: 初始化工具箱配置信息...
【Tools】: 执行工具箱初始化脚本...
【Tools】: 工具箱初始化脚本启动...
【Tools】: 检查环境变量配置
【Tools】: 检查定时任务配置
【Tools】: 检查工具箱开机启动配置
【Tools】: 执行工具箱监控脚本
【Tools】: 防火墙重启插件检查
【Tools】: 运行用户自定义脚本
【Tools】: 工具箱安装完成!
【Tools】: 运行mixbox命令即可配置工具箱
root@XiaoQiang:~# 


```


### 使用 

#### 运行工具
```
# mixbox
获取工具箱插件列表...
***************************************
     *****   MIXBOX 工具箱   *****     
***************************************
当前版本：[0.1.9.13]    最新版本：[0.1.9.13]
设备型号：[R2D]         核心温度：[68°C]
***************************************
00. 退出工具箱
01. 已安装插件
02. 未安装插件
03. 工具箱管理

请输入你的选择：

```

#### 其他
```
mixbox help
```


---------------------------

### FTP
安装ftp：  

```
请输入vsftpd用户名：xxx
请输入vsftpd密码：xxx123456
请输入xxx访问目录：/userdisk/data/ftp

```

创建目录，修改目录权限
```
# mkdir /userdisk/data/ftp
# chmod 777 ftp
```



#### 访问
资源管理器输入：

ftp://xxx.ddns.net

再输入用户名和密码



### Aria2

```
********* Aria2 ***********
[Linux下一款高效的下载工具]
启动aria2服务[1/0] [回车即1]：1
修改aria2端口号()？[1/0] 0
修改aria2配置(空, /userdisk/data/下载)？[1/0] /userdisk/data/aria2Download
请输入aria2外网访问配置[1/0][回车即1]：1
【Aria2】: 正在停止aria2服务... 
【Aria2】: 删除nat-start触发...
【Aria2】: 正在启动aria2服务... 
【Aria2】: 加载aria2配置...
【Aria2】: 更新bt-tracker
【Aria2】: 生成aria2本地web页面
【Aria2】: 添加nat-start触发事件...
【Aria2】: 启动aria2服务完成！
【Aria2】: 访问[http://192.168.31.1/backup/log/aria2]管理服务
【Aria2】: jsonrpc地址:http://192.168.31.1:6800/jsonrpc
```



### FileBrowser
```
********* FileBrowser ***********
[Web文件浏览器]
启动filebrowser服务[1/0] [回车即1]：1
修改filebrowser端口号(1086)？[1/0] 
修改filebrowser默认访问路径(/userdisk/data)？[1/0] 
请输入filebrowser外网访问配置[1/0][回车即1]：0
【FileBrowser】: 正在停止filebrowser服务... 
【FileBrowser】: 删除nat-start触发...
【FileBrowser】: 正在启动filebrowser服务... 
【FileBrowser】: 启动filebrowser服务完成！
【FileBrowser】: 请在浏览器中访问[http://192.168.31.1:1086]，默认用户名密码admin
```

启动有问题，单独启动测试一下：
```
./filebrowser -p 1086 -d /etc/mixbox/apps/filebrowser/bin/filebrowser.db 

daemon /etc/mixbox/apps/filebrowser/bin/filebrowser -a 0.0.0.0 -p 1086 -d /etc/mixbox/apps/filebrowser/bin/filebrowser.db

daemon /etc/mixbox/apps/filebrowser/bin/filebrowser -a 192.168.31.1 -p 1086 -d /etc/mixbox/apps/filebrowser/bin/filebrowser.db
```
修改一下启动脚本：/etc/mixbox/apps/filebrowser/scripts/filebrowser.sh
```
daemon ${mbroot}/apps/${appname}/bin/${appname} -p ${port} -d ${mbroot}/apps/${appname}/config/${appname}.db -l ${mbroot}/var/log/${appname}.log -s $scope 

改成：

daemon ${mbroot}/apps/${appname}/bin/${appname} -a 0.0.0.0 -p ${port} -d ${mbroot}/apps/${appname}/bin/${appname}.db -l ${mbroot}/var/log/${appname}.log  

```


用户名密码改一下：
admin/adminxxx


### aliddns

配置记录：
```
请输入你的选择：1
********* AliDDNS ***********
[动态将你的路由器IP绑定到域名]
启动aliddns服务[1/0] [回车即1]：1
修改aliddns配置？[1/0] 1
请输入aliddns访问ID[回车即xxxxxxxxxxxxxxxxxxxxxxxxx]：
请输入aliddns访问密钥[回车即xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx]：
请输入aliddns域名[例如@.mixbox.com或www.mixbox.com][回车即test.xxxxxxxxxxxxx.top]：@.xxxxxxxxxxxxxxxxx.top    
请输入aliddns检查分钟间隔(建议10)[回车即10]：
支持类型[0仅ipv4/1仅ipv6/2通吃][回车即0]：
重启aliddns服务[1/0][回车即1]：
【AliDDNS】: 正在停止aliddns服务... 
【AliDDNS】: 正在启动aliddns服务... 
【AliDDNS】: 启动aliddns服务完成！

```


https://dns.console.aliyun.com/

> 阿里云的解析速度比较慢，可以买个1块钱的域名试试看



-------------------



### 端口开放相关
/etc/mixbox/bin/base

用的iptable


### 防火墙开放
```
# 查看
iptables -list

# 显示行号
iptables -nL --line-number | more

# 删除INPUT表的第3条

iptables -D INPUT 3

# 允许某个IP访问全部服务
iptables -I INPUT -s 123.240.202.217 -j ACCEPT
```


