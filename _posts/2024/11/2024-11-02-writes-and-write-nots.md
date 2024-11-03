---

title: Nginx 屏蔽恶意域名解析
categories:
- 运维人生
key: f49c35ea-1ac7-11ed-861d-0242ac120111


---

<img src="https://images.animesdata.com/news/2024/11/03/b341-1bf89ce1ba92_2.jpg" width="500" />

最近服务器遭遇恶意域名解析攻击，有人将未经授权的域名解析到我的服务器 IP 上，导致服务器流量异常暴增。百思不得其解为什么会有人这么做，估计是为了绕开 CDN 和 WAF，直接连服务器的 IP，不知道什么原因服务器的 IP 地址被暴露了，也可能是扫描到了。

似乎也没有好的方法在云服务商的 Dashboard 中设置禁止别的域名解析到我的服务器上，因为别人要将域名解析到你的 IP 上，也无法阻止。所以只有考虑在 nginx 的配置上做一下文章。

## 配置详解

将以下配置保存为 `/etc/nginx/conf.d/unauthorized-domains.conf`:

```nginx
# 处理所有未经授权的域名访问请求
# 这个配置块会捕获所有不在其他 server 块中明确定义的域名请求
server {
    # 监听 80 端口(HTTP)
    # default_server 表示这是默认的处理器，当没有其他 server_name 匹配时会走这个配置
    listen 80 default_server;
    listen [::]:80 default_server;
    
    # 监听 443 端口(HTTPS)，直接关闭连接
    listen 443 default_server;
    listen [::]:443 default_server;
    
    # server_name 使用 _ 表示匹配所有域名
    # 这会捕获所有未在其他 server 块中定义的域名
    server_name _;
    
    # 单独记录未授权访问的日志，方便后续分析
    # 建议设置日志轮转，防止日志文件过大
    access_log /var/log/nginx/unauthorized.access.log combined buffer=512k flush=1m;
    error_log /var/log/nginx/unauthorized.error.log warn;
    
    # 关闭不必要的方法
    # 只允许 GET 和 POST 方法
    if ($request_method !~ ^(GET|POST)$ ) {
        return 444;   # 444 是 Nginx 特殊状态码，会直接关闭连接
    }
    
    # 处理所有请求路径
    location / {
        # 返回 403 禁止访问状态码
        # 自定义错误信息，避免暴露服务器信息
        return 403 "Access Denied - Invalid Domain\n";
    }
    
    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
    
    # 禁用 favicon.ico 和 robots.txt 的日志记录
    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }
    
    location = /robots.txt {
        access_log off;
        log_not_found off;
    }
}
```

## 配置说明

1. **端口监听**：
   - 80 端口：处理 HTTP 请求，返回 403 错误
   - 443 端口：由于没有配置证书，直接拒绝 HTTPS 请求

2. **日志配置**：
   - 单独的日志文件便于分析未授权访问
   - 使用缓冲写入提高性能
   - 忽略常见的无用请求日志

3. **安全措施**：
   - 限制 HTTP 方法
   - 阻止访问隐藏文件
   - 自定义错误消息避免信息泄露

## 验证和启用配置

```bash
# 测试配置是否有语法错误
nginx -t

# 如果测试通过，重启 Nginx 服务
systemctl restart nginx
```

## 效果验证

可以使用以下命令测试配置：

```bash
# 测试 HTTP
curl -H "Host: unauthorized-domain.com" http://your-server-ip

# 应该返回: "Access Denied - Invalid Domain"
```

## 日志分析

配置完成后，可以通过以下命令查看未授权访问的情况：

```bash
# 查看访问日志
tail -f /var/log/nginx/unauthorized.access.log

# 统计访问最多的 IP
cat /var/log/nginx/unauthorized.access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10

# 统计访问最多的域名
cat /var/log/nginx/unauthorized.access.log | awk -F '"' '{print $6}' | sort | uniq -c | sort -nr | head -n 10
```

## 监控建议

1. 设置日志轮转，防止日志文件占用过多磁盘空间：

```bash
# 创建日志轮转配置
cat > /etc/logrotate.d/nginx-unauthorized << EOF
/var/log/nginx/unauthorized.access.log /var/log/nginx/unauthorized.error.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 nginx adm
    sharedscripts
    postrotate
        [ ! -f /var/run/nginx.pid ] || kill -USR1 \`cat /var/run/nginx.pid\`
    endscript
}
EOF
```

2. 定期检查访问日志，关注异常情况
3. 考虑配置监控报警，当未授权访问量突增时及时通知

## 额外加固建议

1. 考虑使用 fail2ban 封禁频繁访问的 IP
2. 可以添加 rate limiting 限制请求频率
3. 如果服务器上只有特定的域名在使用，建议在防火墙层面做访问控制


