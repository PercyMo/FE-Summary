## yarn详解

### 一. 常用命令
1. 初始化一个新项目
    ```sh
    yarn init
    ```
2. 安装项目全部依赖
    ```sh
    yarn
    # 或者
    yarn install
    ```
3. 添加依赖包
    ```sh
    yarn add [package]
    yarn add [package]@[version]
    yarn add [package]@[tag]
    ```
4. 将依赖项分别添加到不同的依赖类别中  
    分别添加到 `devDependencies`，`peerDependencies`，`optionalDependencies`
    ```sh
    yarn add [package] -dev
    yarn add [package] -peer
    yarn add [package] -optional
    ```
5. 升级依赖包
    ```sh
    yarn upgrade [package]
    ```
6. 移除依赖包
    ```sh
    yarn remove [package]
    ```
### 二. workspace

```sh
# 安装所有子项目的依赖
yarn install

# 执行某一子项目中的命令
yarn workspace <project> run xx

# 给所有应用都安装依赖
yarn workspaces add package

# 给某个应用安装依赖
yarn workspace <project> add package

# 给根应用安装依赖
yarn add -W -D package

# 重装所有 node_moudles
yarn install --force
```
### 三. 引用
