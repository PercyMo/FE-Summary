## github使用问题

### 一. github头像和用户图片无法显示问题
添加host
```sh
# github 头像
192.30.253.112 Build software better, together 
192.30.253.119 gist.github.com
151.101.184.133 assets-cdn.github.com
151.101.184.133 raw.githubusercontent.com
151.101.184.133 gist.githubusercontent.com
151.101.184.133 cloud.githubusercontent.com
151.101.184.133 camo.githubusercontent.com
151.101.184.133 avatars0.githubusercontent.com
151.101.184.133 avatars1.githubusercontent.com
151.101.184.133 avatars2.githubusercontent.com
151.101.184.133 avatars3.githubusercontent.com
151.101.184.133 avatars4.githubusercontent.com
151.101.184.133 avatars5.githubusercontent.com
151.101.184.133 avatars6.githubusercontent.com
151.101.184.133 avatars7.githubusercontent.com
151.101.184.133 avatars8.githubusercontent.com
# github 内容图片
151.101.184.133 user-images.githubusercontent.com
```

### 二. clone项目 SRA 报错
到[这里](https://www.ipaddress.com/)查下 `github.com` 和 `github.global.ssl.fastly.net`  
host 中修改下对应 ip
```sh
# github SRA报错
140.82.114.3 github.com
199.232.5.194 github.global.ssl.fastly.net
```