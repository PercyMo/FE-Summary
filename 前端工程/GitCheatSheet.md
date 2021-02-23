1. branch分支
```sh
git branch bugFix # 创建一个bugFix新的分支

git checkout bugFix # 切换到新的分支

git checkout -b bugFix-v2 # 创建一个bugFix-v2新分支，并切换到该分支

git fetch -p  # 删除那些远程已经不存在的分支的本地缓存

git reset --soft HEAD~1 # 撤销上次commit，但保留工作区的代码

git checkout . # 本地所有修改的。没有的提交的，都返回到原来的状态

git log --follow ./test.js # 可以追溯文件重命名或者移动前的历史记录
```