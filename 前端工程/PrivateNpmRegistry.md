## npm 私服搭建
近期公司有多个技术团队共享代码包的需求，因此准备搭建 npm 私服，第一时间想到是使用 Verdaccio。但问了运维后发现已经有了 Nexus2，于是把 Nexus2 也折腾了下。

### 一. Nexus2 设置 npm 私服
因为公司已经有了 Nexus2，所以不需要安装，在已有基础上配置即可，整个过程按照下面的参考文章进行，还算顺利。  
[nexus2架设npm私服](https://segmentfault.com/a/1190000016090267)  
文章中对于发布包的描述有问题，不建议参考。  

#### 1. 发布包
1. 先在Nexus2系统界面新增一个用户，设置好权限（这里貌似没什么需要注意的，我就是把所有npm相关的权限都勾选了，粗暴且管用）
2. 终端执行 `npm config edit`
    ```
    email=maobangxin@xx.com // 这里是账号使用的邮箱
    always-auth=true
    _auth=bnBtLWFkbWluOjEyMzQ1Ng==
    ```
    关于`_auth`的值有点玄机，实际上就是 `btoa('username:password')`，用户名密码bse64编码后的值。
3. 发布包  
    ```ah
    npm publish --registry http://nexus.xx.com/nexus/content/repositories/npm-internal/
    ```

#### 2. Nexus2 的问题，最后弃用
全部配置完成后，实际使用发现，无法自己发布 npm 范围包，也无法通过私服安装 npm 范围包。  
```sh
@scope/project-name # /后的名字会被忽略掉，导致报错
```
无法发布范围包还可以忍，但不能安装，导致现有项目无法正常运行，安装过程就各种报错。  
最后确认，Nexus2 不支持 npm 范围包，需要升级至 3.x，放弃了。

### 二. Verdaccio
1. 如何删除已经发布的包  
verdaccio 的本地存储在本地：`.local/share/verdaccio/storage`，可以进入直接删除对应的包即可(手动)

### 三. 使用 Docker 部署 Verdaccio
[使用verdaccio搭建npm私有源](https://mp.weixin.qq.com/s/lTGV7XrsJCvoU3F2bEbovw)  
[Verdaccio 使用 Docker 安装及迁移教程](https://segmentfault.com/a/1190000020684605)  
