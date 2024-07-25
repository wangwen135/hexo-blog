---
title: Nginx-配置文件自动备份脚本
date: 2023-02-18 23:41
tags: 
  - Nginx
categories:
  - [Nginx]
---


脚本：
```
#!/bin/bash

# 配置源目录和备份目录
source_dir="/etc/nginx/conf.d"
backup_dir="/www/mnt/backup/nginx"

# 创建备份目录（如果不存在）
mkdir -p "$backup_dir"

# 获取当前日期（格式：YYYY-MM-DD）
current_date=$(date +%Y-%m-%d)

# 压缩配置文件到临时tar包
tar_filename="/tmp/nginx_config_backup_$current_date.tar.gz"
tar -czf "$tar_filename" -C "$source_dir" .

# 检查上一个备份是否存在且与当前备份不同
last_backup=$(ls -t "$backup_dir" | head -1)
if [ -n "$last_backup" ]; then
    # 解压上一个备份和当前备份
    mkdir -p /tmp/last_backup /tmp/current_backup
    tar -xzf "$backup_dir/$last_backup" -C /tmp/last_backup
    tar -xzf "$tar_filename" -C /tmp/current_backup

    # 使用diff命令比较两个目录中的文件差异
    diff_output=$(diff -r /tmp/last_backup /tmp/current_backup)

    # 清理临时目录
    rm -r /tmp/last_backup /tmp/current_backup

    if [ -z "$diff_output" ]; then
        # 上一个备份与当前备份相同，抛弃当前备份
        echo "No changes in Nginx configuration. Discarding backup."
        rm "$tar_filename"
        exit 0
    fi
fi

# 移动当前备份到备份目录
mv "$tar_filename" "$backup_dir"

# 删除多余的备份（保留最多10个备份）
backup_count=$(ls -1 "$backup_dir" | wc -l)
if [ "$backup_count" -gt 10 ]; then
    oldest_backup=$(ls -t "$backup_dir" | tail -1)
    rm "$backup_dir/$oldest_backup"
fi

echo "Nginx configuration backup completed."
```

编辑定时任务
```
crontab -e
```

增加：
```
5     3       *       *       *       /opt/shell/nginx/nginx_config_backup.sh
```




