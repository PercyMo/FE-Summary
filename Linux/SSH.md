## SSH

### 一. SSH客户端
```bash
# 登录服务器
ssh user@hostname # ssh root@123.45.6.78
```

### 二. scp 命令
scp 相当于 cp 命令 + SSH，主要用于三种复制操作。
* 本地复制到远程
* 远程复制到本地
* 两个远程系统之间的复制
```bash
# 基本语法
scp user@host:foo.txt bar.txt # 将远程主机 user@host 用户主目录下的 foo.txt 复制为本机当前目录的 bar.txt

# 本地复制到远程
# 将本机目录下所有内容拷贝到远程目录下
scp -r localmachine/path_to_the_directory/* username@server_ip:/path_to_remote_directory/
```

### 三. 引用
[SSH 教程](https://wangdoc.com/ssh/index.html)
