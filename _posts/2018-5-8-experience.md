---
title: 搭建Jekyll-NexT类型博客时出现的问题
date: 2018-5-7 18:30:09
categories:
- blog
---
# 1.当我准备好将博客框架以及博文写好之后，我想上传到github上出现错误：
 
```shell
yu-Danger% git push origin master

```
```
remote: Permission to Simpleyyt/jekyll-theme-next.git denied to 2391134843.
fatal: unable to access 'https://github.com/Simpleyyt/jekyll-theme-next.git/': The requested URL returned error: 403

```
### 解决方法：找到项目下面的.git目录中的config文件。然后修改 
```
url=https://github.com/2391134843/2391134843.github.io.git
```
也就是你自己的节点
# 2.当这个问题解决之后还有一个上传问题：
```
  c635cc3..2485178  master -> master
error: update_ref failed for ref 'refs/remotes/origin/master': cannot update the ref 'refs/remotes/origin/master': unable to append to '.git/logs/refs/remotes/origin/master': Permission denied

```
解决方法：

```shell
yu-Danger% sudo git push -f

```
