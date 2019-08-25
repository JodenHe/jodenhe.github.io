---
title: Gitlab 更新 fork 的代码
author: Joden_He
tags: 
  - git
categories: 
  - git
description: 有时候我们在 gitlab fork 了某个项目的代码，然而我们需要怎么更新我们 fork 的代码呢？
---



### 操作步骤

- 先确定是否建立了主仓库的远程数据源:

```shell
git remote -v
```

![remote_rep](/images/6.png)

我们发现只有我们自己的两个源(fetch 和 push)，那这个时候就需要添加主仓库的源。

- 添加主仓库的源

```shell
 git remote add upstream URL
```

> upstream : 随便起的名字
>
> URL： fork的仓库地址
>
> 如果想删除remote的upstream标签，则可以运行： git remote rm upstream

![remote_rep](/images/7.png)

- 更新代码

  ```shell
  git fetch upstream
  git checkout master
  git merge upstream/master
  ```

### 参考

[https://blog.csdn.net/lhh1113/article/details/71038154](https://blog.csdn.net/lhh1113/article/details/71038154)

