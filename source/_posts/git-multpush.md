---
title: Git:如何将代码支持同时上传github和gitee
date: 2020-03-10 10:00:40
categories:
  - [Git]
tag:
    - Git
---

- Github和Gitee的区别主要是服务器位于国外和国内，访问速度有区别

- 那么能将本地代码通过git分别传到两个不同的托管平台吗？能的

- 本文主要介绍如何通过`git remote`设置，将代码分别上传到github和gitee

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.Git支持关联多个远程托管库

- Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。

- 因为git本身是分布式版本控制系统，所以可以同步到多个远程库，所以一个本地库可以既关联GitHub，又关联gitee

- 使用多个远程库时，要注意git给远程库起的默认名称是origin，如果有多个远程库，我们需要用不同的名称来标识不同的远程库

# 2.具体操作

## 2.1 查看远程库信息

`git remote -v`

- 输出结果

```
origin  git@github.com:xxx/LearnGit.git (fetch)
origin  git@github.com:xxx/LearnGit.git (push)
```

## 2.2 删除初始远程库

`git remote rm origin`

## 2.3 重新关联多个远程库

- 关联github远程库

`git remote add github git@github.com:xxx/LearnGit.git`

- 关联gitee远程库

`git remote add gitee git@gitee.com:xxx/LearnGit.git`

> 这样本地的项目就关联了两个不同的远程库，可以通过 git remote -v 进行查看

`git remote -v`

- 输出结果

```
gitee   git@gitee.com:xxx/LearnGit.git (fetch)
gitee   git@gitee.com:xxx/LearnGit.git (push)
github  git@github.com:xxx/LearnGit.git (fetch)
github  git@github.com:xxx/LearnGit.git (push)
```

# 3.拉取代码

> 拉取代码的时候需要声明好关联的哪一个远程库和哪一个分支

- 拉取github远程库的master分支的代码

`git pull github master`

- 拉取gitee远程库的master分支的代码

`git pull gitee master`

# 4.上传代码

> 上传代码同样需要声明好关联的远程库和分支

- 上传代码到github远程库的master分支

`git push github master`

- 上传代码到gitee远程库的master分支

`git push gitee master`

> 通过一推一拉都要点明远程库实现了多库管理的代码同步。

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)









