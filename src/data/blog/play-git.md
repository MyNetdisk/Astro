---
author: Axton
pubDatetime: 2025-07-03T15:22:00Z
modDatetime: 2025-07-03T15:22:00Z
title: 玩转Git三剑客
featured: false
draft: false
tags:
  - React Native
description:
  玩转Git三剑客
---

> 极客时间出品

### 使用Git之前需要做的最小配置
```bash
git init # 创建一个git仓库
```

### 创建local用户信息
```bash
git config user.name "axton.zhou"
git config user.email "axton.zhou@gmail.com"
```

### 给文件重命名的简便方法
```bash
git mv README.txt README.md
```

### 通过git log查看版本演变历史
```bash
git log -1 # 查看最近1条记录
git log -n3 # 查看最近3条记录
git log --oneline # 线性查看，仅显示提交id、commit信息
git log -n3 --oneline # 线性查看最近3条记录
git log --all # 查看所有分支的提交记录
git log --all --graph # 图形化查看所有分支提交记录
git log temp # 查看temp分支的提交记录
git help --web log # 浏览器查看log帮助信息

git branch -v # 命令用于显示当前仓库的所有本地分支及其最新提交信息
git checkout -b temp # 基于当前分支创建temp分支

git commit -am"Add test" # 使用 `-a` 选项的一个常见场景是快速提交代码更改，而无需逐个使用 `git add` 命令
```

### 通过图形界面查看版本历史
```bash
gitk # 图形界面查看版本历史
gitk --all # 图形界面查看所有分支的版本历史
```

### 探秘.git目录
```bash
# config中存储当前项目的用户名、邮箱等配置 
# refs/中存储所有的引用，包括 heads/ 和 tags/ 
# HEAD中存储当前检出的分支或提交的引用 
# objects/存储所有的对象，包括 commit、tree 和 blob
git cat-file -t 3ea572e87 # 查看文件的类型
git cat-file -p 3ea572e87 # 查看文件的内容
```

### 分离头指针情况下的注意事项
```bash
git checkout b41af56c6 # 切换到某个commit，产生分离头指针的情况，没有指向任何分支。
git branch <new-branch-name> f6aa558 # 将分离头指针的commit提交保存到一个新分支。

git diff a11b0249  b41af56c # 两个commit进行比较
git diff HEAD HEAD^1 # 比较当前的commit和上一个commit
git diff HEAD HEAD^1^1 # 比较当前的commit和上上一个commit
```

### 删除分支
```bash
git branch -d fix/readme # 删除fix/readme分支
git branch -D fix/readme # 强制删除fix/readme分支
```

### 修改commit的message
```bash
# 注意修改commit的行为只应该在本地的commit上进行，已经远端仓库的commit不能轻易修改。
git commit --amend # 修改最近一条commit
git rebase -i 8e4e7b79e # reward修改某一条 编辑修改commit的message -i是指交互式的方式 要变更哪一条的commit, commit ID一定要指向它的上一条commit ID（如果是要变更最后一条就必须把最后一条的commit ID写在交互窗口的第一条pick 8e4e7b79e）
git rebase -i a5b4e4805 # squash将连续的几条commit合并成一条
git rebase -i 7ef7814b # 将不连续的commit合并成一个，其实就是将不不联系的commit messag移到一起，前面加一个squash。
```

### 比较差异和恢复
```bash
git diff # 默认比较的是工作区和暂存区的区别
git diff -- index.html # 比较工作区和暂存区的指定文件index.html的差异
git diff --cached # 比较暂存区和HEAD所含文件的差异
git diff temp main # 比较两个分支的差异
git diff temp main -- index.html # 比较两个分支的某个文件index.html的差异
git diff 3ea572e 4562990 -- index.html # 等价于上面这条命令

git reset HEAD # 将变更从暂存区恢复到工作区
git reset HEAD -- styles/styles.css # 将指定的文件变更从暂存区恢复到工作区
git reset HEAD -- hello.md index.html # 将指定的多个文件变更从暂存区恢复到工作区
git checkout -- index.html # 将工作区的变更恢复到暂存区一致

git reset --hard 3ea572e87 # 将HEAD、工作区和暂存区的提交回退到3ea572e87 慎用！！！
git reset --hard HEAD # 恢复到和头指针HEAD一致
```

### 删除文件
```bash
git rm hello.md # 删除文件hello.md
```

### 临时保存当前工作目录中的未提交修改
```bash
git stash # 临时保存未提交修改
git stash list # 查看临时保存的文件列表
git stash apply # 应用列表中第一条临时保存的提交记录，该命令会保留临时列表的第一条记录
git stash pop # 应用列表中第一条临时保存的提交记录，该命令不会保留临时列表的第一条记录
```