## Jenkins自动化部署前端项目

### 一. Jenkins安装
阿里云ECS服务器，CentOS  

**1. 登录服务器更新系统软件**  
```sh
$ yum update
```

**2. 安装java和git**  
```sh
$ yum install java
$ yum install git
```

**3. 安装nginx**  
```sh
$ yum install nginx // 安装
$ service nginx start // 启动
```
出现`Redirecting to /bin/systemctl start nginx.service`，说明启动成功。

**4. 安装jenkins**  
```sh
$ wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key

$ yum install jenkins // 完成之后直接使用 yum 命令安装 Jenkins

$ service jenkins restart // 启动 jenkins
```

1. jenkins启动成功后默认是8080端口，浏览器输入服务器ip + 8080看到jenkins首页。  
2. 输入`cat /var/lib/jenkins/secrets/initialAdminPassword`查看初始密码。  
3. 选择推荐通用插件安装，等待安装完成后，初始化账户  
    ![jenkins安装插件](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/01.png?x-oss-process=image/resize,w_500)

    ![jenkins安装插件ing](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/02.png?x-oss-process=image/resize,w_500)

    ![jenkins创建管理员账号](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/03.png?x-oss-process=image/resize,w_500)

4. 安装两个推荐的插件Rebuilder和SafeRestart  
5. 安装node.js插件  
    系统管理 -> 全局工具配置 -> NodeJs配置项  

    配置一个node版本  

    ![NodeJs插件配置](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/05.png?x-oss-process=image/resize,w_500)

### 二. 自动化构建部署项目
**1. 创建任务**  
1. 创建一个新任务  
2. jenkins关联Github项目地址  
    ![jenkins关联Github项目地址](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/06.png?x-oss-process=image/resize,w_500)

    **这里有个小插曲，构建控制台显示 `git fetch` 超时，任务始终卡在这里**
    ```
    +refs/heads/*:refs/remotes/origin/* # timeout=10 ERROR: Error cloning remote repo 'origin' hudson.plugins.git.GitException: Command "git fetch --no-tags --progress -- https://git.xx.com.xx/xx/xxx-xxx.git +refs/heads/*:refs/remotes/origin/*" returned status code 128: stdout:  stderr: remote: Enumerating objects: 562, done
    ```
    **解决：**  
    1. 任务配置 -> 源码管理 -> 新增高级克隆行为，克隆和可拉取操作的超时时间设置为30min，勾选浅克隆，浅克隆深度1  
    2. 任务配置 -> 源码管理 -> 新增高级检出行为，检出操作的超时时间设置为30min  
    ![git fetch 超时](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/08.png?x-oss-process=image/resize,w_500) 


3. 选择构建环境并编写shell命令  
    ![选择构建环境并编写shell命令](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/07.png?x-oss-process=image/resize,w_500)  

配置完成后点击立即构建，等待构建完成，点击工作空间可以发现已经多出一个打包后的dist目录。点击控制台输出可以查看详细构建log。  
到这里已经实现了本地代码提交github，然后在jenkins上点击构建，可以拉取代码并打包

**2. 安装Publish Over SSH插件，实现服务器自动部署**  
1. 安装完成后设置服务器信息  
    系统管理 -> 系统配置 -> Publish over SSH  

    填写服务器信息，设置账号密码，填完点击test，出现success说明配置成功
    ```
    Hostname: 需要连接ssh的主机名或ip地址（推荐ip）
    ```

    ![Publish over SSH](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/09.png?x-oss-process=image/resize,w_500) 

2. 工程中配置**构建后操作**  
    send build artificial over SSH，参数说明：
    ```
    Name: 选择一个配置好的服务器
    Source files: 要传输的文件路径
    Remove prefix: 要去掉的前缀，不写远程服务器的目录结构将和source files写的一致
    Remote directory: 写要部署在远程服务器那个地址下，不写就是SSH Services配置里默认远程目录
    Exec commond: 传输完了要执行的命令，这里执行了进入test目录，解压缩，解压缩完成后删除压缩包三个命令
    ```
    注意在构建中添加压缩dist目录命令  

    ![构建后操作](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/10.png?x-oss-process=image/resize,w_500)

    执行构建，发现目标服务器test目录下有了要运行的文件。

**3. push代码到github，触发webhook，jenkins自动执行构建**  
1. jenkins安装Generic Webhook Trigger插件，工程配置 -> 构建触发器 -> 选择Generic Webhook Trigger，填写token  
    ![Generic Webhook Trigger](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/11.png?x-oss-process=image/resize,w_500)

2. github配置webhook  
    选择github项目 -> settings -> webhooks -> add webhook  

    配置jenkins服务器地址 + token参数，选择push代码时触发webhook  

    ![github配置webhook](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Jenkins/12.png?x-oss-process=image/resize,w_500)

### 三. 总结

### 四. 引用
[让Jenkins自动布署你的Vue项目](https://mp.weixin.qq.com/s/2MFpJr32hVy0HBK9Y_WOqQ)