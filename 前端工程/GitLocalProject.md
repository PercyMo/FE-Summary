## 本地项目上传git

### 一. 命令
```sh
git init
git add .
git commit -m '备注'
git remote add origin 仓库地址
git push -u origin master
```

```sh
# 连接到远程仓库，并为该仓库创建别名
git remote add origin https://github.com/CnPeng/MyCustomAlertDialog.git
```

### 二. 排错
如果执行`git push -u origin master`报错，是因为git上的项目不是空的，大部分情况是因为有一个`README.md`文件。
```sh
error: failed to push some refs to 'git@github.com:xxxxxxxxxxx.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
解决方法：
```sh
git pull --rebase origin master
git push -u origin master
```