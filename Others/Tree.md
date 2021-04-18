## tree生成文件目录树

1. [Mac下的 tree 命令 输出目录树层结构](https://www.jianshu.com/p/9411d60950bf)  
2. [tree命令-一键生成目录结构](https://juejin.cn/post/6844903736016371725#heading-17)

2. **基本使用**  
    ```sh
    cd 到指定目录
    tree --help
    tree -d                     // 只显示文件夹
    tree -I "node_modules"      // 过滤文件夹
    tree -L 2 >README.md        //导出 README.md 文件
    ```

3. tree -P pattern
仅列出与通配符模式匹配的文件。
```sh
tree -P *.css
```
```sh
.
└── test
    ├── css
    │   ├── jquery-ui.css
    │   └── main.css
    ├── img
    │   └── head
    └── js
```