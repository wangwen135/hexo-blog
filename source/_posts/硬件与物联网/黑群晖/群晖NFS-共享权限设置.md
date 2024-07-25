---
title: 群晖NFS-共享权限设置
date: 2020-03-02 23:41
tags: 
  - 黑群晖
categories:
  - [硬件与物联网, 黑群晖]
---


参考：  
https://kb.synology.cn/zh-cn/DSM/tutorial/What_can_I_do_to_access_mounted_folders_NFS

### 设置
1、开启服务
控制面板 -> 文件服务 -> SMB/AFP/NFS -> 勾选 启用NFS服务  
> 为保证兼容性，建议勾选‘启用NFS v4.1支持’

2、设置共享文件的NFS权限  
控制面板 -> 共享文件夹 -> 选择需要共享的文件 -> 编辑 -> NFS权限 -> 新增  

- 服务器名称或IP地址：* 
> * 全部IP可以访问，也可以指定IP段等，如：192.168.31.1/24
- 权限：可读写
- Squash：映射所有用户为 admin  （下面说明）
- 安全性：sys

> 应用 NFS 权限后，您可在 NFS 权限选项卡的左下角找到共享文件夹的装载路径。装载路径应采用以下格式： /[volume name]/[shared folder name]

### 允许访问所有用户

如果要向所有用户授予相同权限，请设置**Squash** 选择每个文件/文件夹的NFS规则并选择将所有用户映射到admin 。

当使用此Squash选项设置NFS权限时，所有用户将被视为Synology NAS上的“管理员”并有权访问所有文件/文件夹。  
当用户创建文件/文件夹时，文件/文件夹的创建者被列为“admin”。

### 向不同的用户提供不同的访问权限

如果您要为不同的用户提供不同的访问权限，您必须将所有计算机和Synology NAS加入同一个LDAP服务器。为Synology NAS 1上的每个文件/文件夹设置LDAP帐户权限，以便不同用户（LDAP帐户）可以通过相应权限访问文件/文件夹。然后，参阅本文以为每个文件/文件夹设置NFS规则，并为Squash选择**无映射**。


> 注：  
> UID编号为0-125之间，仅供Synology NAS中的本地用户使用。若要设置LDAP UID，请使用大于1025的数字。


### 访问

参考：  
https://kb.synology.cn/zh-cn/DSM/tutorial/How_to_access_files_on_Synology_NAS_within_the_local_network_NFS

#### 安装软件

##### Ubuntu
```
sudo apt update
sudo apt install nfs-common
```

##### CentOS/Redhat/Fedora
```
sudo yum install nfs-utils
```

#### 挂载目录
查看挂载目录
```
showmount -e 192.168.31.66

```


输入挂载命令以在客户端通过 NFS 装载共享文件夹

```
sudo mount -t nfs [Synology NAS IP address]:[mount path of shared folder] /[mount point on NFS client]


示例：
sudo mount -t nfs 196.168.x.x:/volumeX/test /mnt
mount -t nfs 192.168.31.66:/volume2/video /mnt/video
```

#### 查看
输入disk free命令以确认您已成功装载共享文件夹。文件系统列中的输出应采用以下格式： [Synology NAS IP address]:[mount path of shared folder]

```
df -h

192.168.31.66:/volume2/video  3.5T   68G  3.5T    2% /mnt/video
```


### 其他
挂载不了时，检查一下命令，地址  
ping一下ip  
telnet一下端口： telnet 192.168.31.66 2049
