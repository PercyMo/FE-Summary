## Snippets代码片段

### 一. 使用
> VS Code为例，入口在 code -> 首选项 -> 用户片段

代码片段分为两种：
1. 全局代码片段（每种语言环境下都能触发的代码块）
2. 对应语言的局部代码片段（只能在对应语言环境下才能触发）

![vs code 代码片段](http://img.vanilla.ink/me/webproject/FE-Summary/Others/Snippets/01.png)


### 二. Snippet语法
```json
"console.log": {
    "prefix": "log",
    "body": [
        "console.log($1)",
        "$2"
    ],
    "description": "console.log快捷"
  }
```
* `console.log`对应代码片段名称
* `prefix`对应触发代码片段的字符
* `body`对应代码片段内容，可以是字符串，也可以为数组。
* `description`对应代码片段描述

1. **占位符 $**  

`$`后边紧跟数字可指定代码片段触发落入编辑器之后的光标位置。

2. **占位内容的可选项**  
```json
"方法注释": {
    "prefix": "zs-Function",
    "body": [
        "/**",
        " * @param name... { ${1|Boolean,Number,String,Object,Array|} }",
        " * @description description...",
        " * @return name... { ${2|Boolean,Number,String,Object,Array|} }",
        " */",
        "$0"
    ],
    "description": "添加方法注释"
}
```
![占位内容的可选项](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/Others/Snippets/02.png)

3. **变量**  
使用`$name`或者`${name:default}`可以插入变量的值。
    1. 文档相关  

        | 变量 | 变量含义 |
        |:- |:- |
        | `TM_SELECTED_TEXT` | 当前选定的文本或空字符串 |
        | `TM_CURRENT_LINE` | 当前行的内容 |
        | `TM_CURRENT_WORD` | 光标下的单词内容或空字符串 |
        | `TM_LINE_INDEX` | 基于零索引的行号 |
        | `TM_LINE_NUMBER` | 基于单索引的行号 |
        | `TM_FILENAME` | 当前文档的文件名 |
        | `TM_FILENAME_BASE` | 当前文档没有扩展名的文件名 |
        | `TM_DIRECTORY` | 当前文档的目录 |
        | `TM_FILEPATH` | 当前文档的完整文件路径 |
        | `CLIPBOARD` | 剪贴板的内容 |
        | `WORKSPACE_NAME` | 已打开的工作空间或文件夹的名称 |

    2. 当前日期和时间  

        | 变量 | 变量含义 |
        |:- |:- |
        | `CURRENT_YEAR` | 当前年份 |
        | `CURRENT_YEAR_SHORT` | 当前年份的最后两位数 |
        | `CURRENT_MONTH` | 月份为两位数（例如'02'） |
        | `CURRENT_MONTH_NAME` | 月份的全名（例如'June'）（中文语言对应六月） |
        | `CURRENT_MONTH_NAME_SHORT` | 月份的简称（例如'Jun'）（中文语言对应是6月） |
        | `CURRENT_DATE` | 这个月的哪一天 |
        | `CURRENT_DAY_NAME` | 当天是星期几（例如'星期一'） |
        | `CURRENT_DAY_NAME_SHORT` | 当天是星期几的简称（例如'Mon'）（中文对应周一） |
        | `CURRENT_HOUR` | 24小时时钟格式的当前小时 |
        | `CURRENT_MINUTE` | 当前分 |
        | `CURRENT_SECOND` | 当前秒 | 

    3. 要插入行或者块注释  

        | 变量 | 变量含义 |
        |:- |:- |
        | `BLOCK_COMMENT_START` | 输出：PHP /*或HTML格式<!-- |
        | `BLOCK_COMMENT_END` | 输出：PHP */或HTML格式--> |
        | `LINE_COMMENT` | 输出：PHP //或HTML格式 |

### 三、引用
[VSCode 利用 Snippets 设置超实用的代码块](https://juejin.im/post/5d0496415188257fff23b077)
