---
title: letsencrypt 证书申请和自动续期
date: 2024-05-19 22:15
tags: 
  - SSL
  - 证书
  - Let’s Encrypt
categories:
  - [网络与安全, SSL]
---


Let’s Encrypt 是一家证书颁发机构。
推荐使用 Certbot ACME 客户端，Certbot 是 Let's Encrypt 提供的官方工具，用于自动化证书的申请和更新。

https://certbot.eff.org/instructions?ws=nginx&os=centosrhel7&tab=wildcard

https://eff-certbot.readthedocs.io/en/latest/install.html

### Let's Encrypt 的主要特点：
- **免费：** 任何个人或组织都可以免费申请和使用 Let's Encrypt 的证书（包括通配符证书）。
- **自动化：** 通过工具（如 certbot）可以自动生成、验证、颁发、安装和更新证书，减少了手动管理的复杂性。
- **短生命周期：** Let's Encrypt 证书的有效期为 90 天，建议用户在证书到期前 30 天内更新。自动化工具如 certbot 可以帮助定期自动更新证书。
- **广泛兼容：** Let's Encrypt 证书在现代浏览器中得到广泛支持，可以被大多数用户信任。

---

### 安装环境说明
**CentOS Linux release 7.9**  
**nginx version: nginx/1.20.x**  


### 安装Certbot
https://eff-certbot.readthedocs.io/en/latest/install.html

> 不使用 snapd 安装，太麻烦了，直接使用yum安装

```
yum install certbot
```


### 手动获取通配符证书
使用DNS认证，申请通配符证书

```
sudo certbot certonly --manual --preferred-challenges dns -d wangwen135.top -d '*.wangwen135.top'
```
![1715755367252.png](https://img.wangwen135.top:23456/note/2024/05/66445969f3959.png)

输入邮箱地址，同意条款等，然后就需要去域名解析那边配置一个txt记录


![1715755617357.png](https://img.wangwen135.top:23456/note/2024/05/66445a6422486.png)

确保解析生效：
```
nslookup -type=txt _acme-challenge.wangwen135.top
```

然后回到控制台，继续

然后提示还需要再添加一条，重复上面的操作，不要删除之前的，名字相同是可以的

```
>nslookup -type=txt _acme-challenge.wangwen135.top
服务器:  dns4.xxx.cn
Address:  10.10.66.12

非权威应答:
_acme-challenge.wangwen135.top  text =

        "GBp_6iDCPE44wOkjTHwxUvlFzDNgwbZV0pRF5MDhpEw"
_acme-challenge.wangwen135.top  text =

        "aVWrn3qBdTBIz5GW6JRwF8C858gE5N3r_8MQljYjfhw"
```

回车生成成功
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Resetting dropped connection: acme-v02.api.letsencrypt.org
Cleaning up challenges
Subscribe to the EFF mailing list (email: wangwen135@gmail.com).
Starting new HTTPS connection (1): supporters.eff.org

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/wangwen135.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/wangwen135.top/privkey.pem
   Your certificate will expire on 2024-08-13. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

生成的证书文件在：/etc/letsencrypt/live/wangwen135.top 目录中

> **不要移动或者重命名生成的文件**


----

### 使用证书

直接在nginx中配置，如：
```
server {
    listen 8443 ssl http2;
    server_name wangwen135.top home.wangwen135.top;

    ssl_certificate /etc/letsencrypt/live/wangwen135.top/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wangwen135.top/privkey.pem;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
	...
	...
```


----

### 自动续签

使用 Let's Encrypt 的 --manual 模式进行证书验证时，需要在每次续期时手动在 DNS 解析中添加 TXT 记录。因为 --manual 模式要求用户手动完成域名所有权的验证过程。

Let's Encrypt 官方提供了一些 DNS 插件来支持不同的 DNS 提供商，这些插件可以帮助自动化 DNS 记录的管理，可以通过 API 自动更新 DNS 记录。
如：certbot-dns-cloudflare、certbot-dns-google 等。

但是官方提供的 Certbot 客户端没有直接支持阿里云 DNS API 的插件。

在github上找了一个`certbot-dns-alidns` 这个插件可以用于与阿里云 DNS API 集成，实现自动化证书续期。

https://github.com/justjavac/certbot-dns-aliyun


#### 安装阿里云CLI命令行工具
```
wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz
tar xzvf aliyun-cli-linux-latest-amd64.tgz
sudo cp aliyun /usr/local/bin
rm aliyun
```

安装完成后需要配置凭证信息

准备key：
```
AccessKey ID:  
LTAI5txxxxxxxxxxx

AccessKey Secret:  
A0ooBtxxxxxxxxxxxxxxxxxxxxxxxx
```
> 上阿里云上操作

执行：aliyun configure
```
[root@centos7-85 ~]# aliyun configure
Configuring profile 'default' in 'AK' authenticate mode...
Access Key Id [*********************kvh]: 
Access Key Secret [***************************zVy]: 
Default Region Id []: cn-hangzhou
Default Output Format [json]: json (Only support json)
Default Language [zh|en] zh: 
Saving profile[default] ...Done.

Configure Done!!!
..............888888888888888888888 ........=8888888888888888888D=..............
...........88888888888888888888888 ..........D8888888888888888888888I...........
.........,8888888888888ZI: ...........................=Z88D8888888888D..........
.........+88888888 ..........................................88888888D..........
.........+88888888 .......Welcome to use Alibaba Cloud.......O8888888D..........
.........+88888888 ............. ************* ..............O8888888D..........
.........+88888888 .... Command Line Interface(Reloaded) ....O8888888D..........
.........+88888888...........................................88888888D..........
..........D888888888888DO+. ..........................?ND888888888888D..........
...........O8888888888888888888888...........D8888888888888888888888=...........
............ .:D8888888888888888888.........78888888888888888888O ..............
```


#### 安装 certbot-dns-aliyun 插件

```
git clone git@github.com:justjavac/certbot-dns-aliyun.git
cd certbot-dns-aliyun/
sudo cp alidns.sh /usr/local/bin
sudo chmod +x /usr/local/bin/alidns.sh
sudo ln -s /usr/local/bin/alidns.sh /usr/local/bin/alidns

```

#### 申请证书

测试是否能正确申请：

```
certbot certonly -d wangwen135.top -d *.wangwen135.top --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run

```

正式申请时去掉 --dry-run 参数：

```
certbot certonly -d wangwen135.top -d *.wangwen135.top --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" 

```

#### 证书续期

```
certbot renew --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run

```
如果以上命令没有错误，把 `--dry-run` 参数去掉。


#### 自动续期

添加定时任务 crontab。
```
sudo crontab -e
```
> 以root身份执行

输入

```
1 1 */1 * * certbot renew --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --deploy-hook "nginx -s reload" >> /var/log/letsencrypt/c
rontab/certbot_renew.log 2>&1

```

> 上面脚本中的 --deploy-hook "nginx -s reload" 表示在续期成功后自动重启 nginx。

创建日志目录：  
```
mkdir -p /var/log/letsencrypt/crontab/
```

日志文件位于：
- /var/log/letsencrypt/letsencrypt.log
- /var/log/letsencrypt/crontab/certbot_renew.log 

**certbot renew** 命令会检查所有已安装的证书，并在证书的有效期少于 30 天时尝试更新它们。如果证书在此期间成功更新，新的有效期将从更新的日期开始重新计算。



---

---




### 调试


crontab 可能没有正常执行，查看记录：
```
cat /var/log/cron
```

查看任务执行日志
```
tail -33f /var/log/letsencrypt/crontab/certbot_renew.log 
```
将定时任务调整为：
```
*     *      *        *       *       certbot renew....
```
> 每分钟一次，确保没有什么异常
