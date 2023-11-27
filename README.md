# nginx-log 收集nginx日志

### 配置nginx.conf
```shell
log_format json escape=json '{"timestamp":$msec,'
          '"remote_addr":"$remote_addr",'
          '"nginx_host":"$server_addr",'
          '"http_host":"$http_host",'
          '"ssl_protocol":"$ssl_protocol",'
          '"ssl_cipher":"$ssl_cipher",'
          '"request":"$request",'
          '"request_uri":"$request_uri",'
          '"referer":"$http_referer",'
          '"http_user_agent":"$http_user_agent",'
          '"request_length":$request_length,'
          '"request_method":"$request_method",'
          '"status":$status,'
          '"body_bytes_sent":$body_bytes_sent,'
          '"request_time":$request_time,'
          '"upstream_addr":"$upstream_addr",'
          '"upstream_response_time":$upstream_response_time}';
```
### 配置并启动redis服务
```shell
127.0.0.1:6379
```

### 配置并启动elasticsearch
```shell
http://127.0.0.1:9200/
```

### 配置文件：config.json
```json
{
  "domain": "ha666.com",
  "all_in": true,
  "cache_second": 86400,
  "data": {
    "redis": {
      "addr": "127.0.0.1:6379",
      "password": ""
    }
  },
  "source": {
    "type": "file",
    "file_path": "/var/log/nginx/ha666.com.access.log"
  },
  "target": {
    "type": "elasticsearch",
    "elastic_search": {
      "endpoint": "http://127.0.0.1:9200/",
      "index_rule": "nginx-access-<yyyy-mm>"
    }
  }
}
```

### 配置service：
```shell
# cat nginx-log.service
[Unit]
Description=nginx-log
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=simple
User=root
Group=root
Restart=always

# Prevent writes to /usr, /boot, and /etc
ProtectSystem=full

# Doesn't yet work properly with SELinux enabled
# NoNewPrivileges=true

PrivateDevices=true

WorkingDirectory=/root/proj/nginx-log
ExecStart=/root/proj/nginx-log/nginx-log

KillMode=process
KillSignal=SIGTERM

# Don't want to see an automated SIGKILL ever
SendSIGKILL=yes

RestartSec=20s
UMask=007
```

### 启动、停止、重启服务
```shell
systemctl start nginx-log.service
systemctl stop nginx-log.service
systemctl start nginx-log.service
```
