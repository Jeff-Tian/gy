---
layout: post
title:  "Windows 下 Ubuntu 子系统 git push 报 ssh key permission too open 错误"
date:   2018-08-05 20:40:06 +0800
categories: 程序员日常
---

### 症状
我在 `Windows 10` 下安装了 `Ubuntu` 子系统，通过 `bash` 做 `git` 提交时，报错说 `/home/user/.ssh/id_rsa` 文件的权限为 `0777`，太开放了，不能用。从而引发代码提交失败。

### 失败的尝试
```
chmod 600 /home/user/.ssh/id_rsa
```
然后再提交，发现还是报同样的错误，也就是说，以上命令改不了 `id_rsa` 文件的权限。

### 分析
切换到 `id_rsa` 文件的目录：
```
cd /home/user/.ssh
ll
```
发现 `id_rsa` 指向了 `/mnt/c/Users/user/.ssh/id_rsa`，而后者是在 `Windows` 文件系统里，所以很有可能 chmod 改不了 `Windows` 主机的文件权限。

### 验证一下
如果在 `/home/user/.ssh` 目录下新建一个文件，通过 `chmod` 是可以修改其权限的。于是说明以上猜测能站住脚。

### 解决方案
删除那个默认的快捷方式：
```
rm id_rsa
rm id_rsa.pub
```
然后新建一个 `ssh key`：
```
sshkey-gen
```
将新建的 `ssh public key` 打印出来：
```
cat id_rsa.pub
```
将其复制，并粘贴至源代码库的设置 `ssh key` 的页面，保存。

然后再重新 `git push`，成功！
