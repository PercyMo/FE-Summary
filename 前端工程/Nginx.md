## Nginx

### 一. 介绍
#### 1. 安装
```sh
# mac 安装 nginx
brew install nginx

# 查看版本号
nginx -v
```

#### 2. 常用命令
```sh
# 启动
nginx

# 测试配置文件是否正确
nginx -t

# 重启 nginx，修改配置后重新加载配置
nginx -s reload

# 快速停止 nginx
nginx -s stop

# 完整有序地停止 nginx
nginx -s quit
```

### 二. 配置
1. mac  
    `/usr/local/etc/nginx/nginx.conf`
2. linux 下查看实际调用的配置文件  
    ```sh
    [root@b2cadm1294 ~]# ps -ef | grep nginx
    root     23808 23611  0 16:34 pts/0    00:00:00 grep nginx
    root     31017     1  0 00:01 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    apache   31018 31017  0 00:01 ?        00:00:00 nginx: worker process
    apache   31019 31017  0 00:01 ?        00:00:00 nginx: worker process
    ```
### 三. 反向代理

### 四. 负载均衡

### 五. 静态文件服务
1. nginx 配置
    ```sh
    # nginx.conf

    http {
        server {
            listen 80;
            location / {
                autoindex on;
                root /Users/posy/www;
            }
        }
    }
    ```

2. 将文件放置在 `/Users/posy/www` 目录下，访问 `localhost:8080`，就可以看到实际效果了

### 六. 引用