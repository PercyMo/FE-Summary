## git commit

### 一. 约定提交信息结构
```
<type>[optional scope]: <subject>
//空一行
[optional body]
//空一行
[optional footer(s)]
```

### 二. 约定 commit type
* feat：增加一个新功能
* fix：修复bug
* docs：只修改了文档
* style：做了不影响代码含义的修改，空格、格式化、缺少分号等等
* refactor：代码重构，既不是修复bug，也不是新功能的修改
* build: 构建流程、外部依赖变更，比如升级 npm 包、修改 webpack 配置等
* perf：改进性能的代码
* test：增加测试或更新已有的测试
* chore：构建或辅助工具或依赖库的更新

### 三. commit 生成工具
commitizen/cz-cli：命令行工具  
cz-conventional-changelog：一个符合Angular团队规范的 preset  
1. 项目级安装（推荐）  
    ```sh
    yarn add commitizen cz-conventional-changelog --dev
    ```
    package.json中配置：
    ```json
    "script": {
        ...,
        "commit": "git-cz"
    },
    "config": {
        "commitizen": {
            "path": "node_modules/cz-conventional-changelog"
        }
    }
    ```

2. 全局安装  
    ```sh
    npm install -g commitizen cz-conventional-changelog
    echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
    ```

之后，你可以在项目中使用`git cz`或`yarn commit`提交代码。  

### 四. 自动版本管理和生成 CHANGELOG
* 自动修改最新版本号，可以是`package.json`或者自定义一个文件
* 读取最新版本号，创建一个新的`git tag`
* 根据提交信息，生成`CHANGELOG.md`
* 创建一个新提交包括`CHANGELOG.md`和`package.json`

#### 1. 项目安装
```sh
yarn add standard-version --dev
```
#### 2. 添加`npm script`
```json
{
  "scripts": {
    "release:major": "standard-version --release-as major --no-verify", // 修改副版本号
    "release:patch": "standard-version --release-as patch --no-verify", // 修订版本号
  }
}
```
#### 3. 声明周期
* `prerelease`:所有脚本执行之前
* `prebump`/`postbump`:修改版本号前后
* `prechangelog`/`postchangelog`:生成changelog前后
* `pretag`/`posttag`:生成tag标签前后

`standard-version`本身只针对本地，并没有push操作，可以在最后一步生成tag后，执行push操作。
```json
// package.json
"standard-version": {
  "scripts": {
    "posttag": "git push --follow-tags origin HEAD"
  }
}
```

### 五. 引用
[优雅的提交你的 Git Commit Message](https://juejin.im/post/5afc5242f265da0b7f44bee4)

[规范GIT代码提交信息&自动化版本管理](https://jelly.jd.com/article/5f51aa34da524a0147e9529d)

[让你的 commit 更有价值(规范)](https://mp.weixin.qq.com/s/n2eOUfPEJh_IGT3hyQNhAQ)
