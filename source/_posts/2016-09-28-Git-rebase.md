---
title: Git rebase
date: 2016-09-28 11:17:46
category: Git
tags: 
    - Git
---

将目标分支合并到当前分支
```sh
git rebase developer
```

将远程分支合并到当前分支
```sh
git rebase origin/developer
```

如果遇到冲突，rebase中断，解决冲突，使用
```sh
git add .
```
添加修改，然后
```sh
git rebase --continue
```
或者放弃rebase
```sh
git rebase --abort
```
