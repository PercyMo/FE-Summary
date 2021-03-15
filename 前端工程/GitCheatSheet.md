## git基本操作

### 一. Git 基础
#### 1. branch分支
```sh
git fetch -p  # 删除那些远程已经不存在的分支的本地缓存

git log --follow ./test.js # 可以追溯文件重命名或者移动前的历史记录

git show <commitId> # 查看某一次历史提交的改动记录

git tag -a v0.1.0 -m "my version 0.1.0" # 打tag标签
```
#### 2. 撤销操作
```sh
# 本地所有修改的。没有的提交的，都返回到原来的状态
git checkout .

# 撤销上次commit，但保留工作区的代码
git reset --soft HEAD~1

# 撤销中间某次已提交的 commit
git revert <commitHash>
```

### 二. Git 分支
#### 1. 分支新建与合并
```sh
# 创建一个bugFix新的分支
git branch bugFix

# 切换到新的分支
git checkout bugFix

# 创建一个bugFix-v2新分支，并切换到该分支
git checkout -b bugFix-v2
```

### 三. Git 高级用法
#### 1. cherry-pick
```sh
a - b - c - d   Master
        \
        e - f - g Feature
```
将 feature 分支的某次 commit，应用到 master
```sh
git checkout master
git cherry-pick <f:commitHash>
```
操作完成后
```sh
a - b - c - d - f   Master
        \
        e - f - g Feature
```