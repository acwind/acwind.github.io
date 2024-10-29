---

title: 如何在 Crontab 中使用变量
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120110


---

在使用Linux的定时任务管理工具crontab时，合理使用变量可以让我们的任务配置更加灵活和易于维护。

## 1. 环境变量设置

在crontab中，我们可以直接定义常用的环境变量：

```bash
# 设置基本环境变量
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=your@email.com
HOME=/home/username

# 使用定义的变量
0 * * * * $HOME/scripts/backup.sh
```

## 2. 日期时间变量

日期时间是crontab中最常用的变量之一。以下是几种常用的日期格式：

```bash
# YYYY-MM-DD格式
0 0 * * * echo "Backup $(date +\%Y-\%m-\%d)" >> /var/log/backup.log

# YYYYMMDD格式
0 0 * * * tar -czf backup_$(date +\%Y\%m\%d).tar.gz /path/to/backup

# 年月日时分秒完整格式
0 0 * * * echo "Backup started at $(date +\%Y-\%m-\%d_\%H:\%M:\%S)" >> /var/log/backup.log
```

注意：在crontab中使用`%`符号时需要用反斜杠`\`转义。

## 3. 加载外部环境变量

有时我们需要使用用户配置文件中定义的环境变量，可以这样做：

```bash
# 方法1：使用source命令
0 * * * * . $HOME/.profile; /path/to/script.sh

# 方法2：使用bash -c
0 * * * * /bin/bash -c 'source $HOME/.bashrc; /path/to/script.sh'
```

## 4. 在脚本中使用变量

对于复杂的任务，建议创建单独的脚本文件，例如`backup.sh`：

```bash
#!/bin/bash

# 定义变量
DATE=$(date +%Y-%m-%d)
BACKUP_DIR="/backup/$DATE"
LOG_FILE="/var/log/backup_${DATE}.log"

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 执行备份
tar -czf "$BACKUP_DIR/backup_${DATE}.tar.gz" /path/to/source 2>> "$LOG_FILE"

# 删除30天前的备份
find /backup -type d -mtime +30 -exec rm -rf {} \;
```

然后在crontab中调用这个脚本：

```bash
0 0 * * * /path/to/backup.sh
```

## 5. 常用示例

### 5.1 按时间归档日志

```bash
# 每天零点将日志移动到对应日期的目录
0 0 * * * mkdir -p /logs/$(date +\%Y)/$(date +\%m) && mv /var/log/*.log /logs/$(date +\%Y)/$(date +\%m)/
```

### 5.2 清理旧文件

```bash
# 删除30天前的文件
0 0 * * * find /path/to/logs -name "log_$(date -d '30 days ago' +\%Y\%m\%d)*" -delete
```

### 5.3 创建带时间戳的备份

```bash
# 每小时创建一次备份
0 * * * * mysqldump -u root -p'password' database > /backup/db_$(date +\%Y\%m\%d_\%H\%M).sql
```

## 6. 注意事项

1. **路径问题**：建议使用绝对路径，避免使用相对路径。

2. **转义字符**：在crontab中，百分号`%`需要用反斜杠`\`转义。

3. **环境变量**：crontab的环境变量与用户登录shell时的环境变量不同，需要特别注意。

4. **日志记录**：建议为重要的定时任务添加日志记录：
```bash
0 0 * * * /path/to/script.sh >> /var/log/script.log 2>&1
```


