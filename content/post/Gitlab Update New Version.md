---
title: "Gitlab Update New Version"
date: 2019-04-22T11:17:40+08:00
tags: [“Gitlab”]
categories: [“Gitlab”]
draft: false
---

接触的这段时间gitlab不断的进行迭代更新，所以如果有实用的新功能或严重的bug修复时，必然要考虑gitlab的更新。

## 一、下载新版本的RPM包
### 途径1：通过清华开源镜像站
```
# cat /etc/yum.repos.d/gitlab-ce.repo 
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

### 途径2：从官方获取RPM包后上传到/root目录下 
官方下载：https://packages.gitlab.com/gitlab/gitlab-ce/

### 途径3：直接yum update gitlab-ce即可
```
# yum list all |grep gitlab-ce*
gitlab-ce.x86_64                     11.2.1-ce.0.el6          @gitlab-ce        
gitlab-ce.x86_64                     11.2.3-ce.0.el6          gitlab-ce  
```

##二、更新gitlab
### 2.1 关闭部分gitlab服务
```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-ctl stop nginx
```

### 2.2 升级
`yum  -y update gitlab-ce`

### 2.3 重新配置gitlab
gitlab-ctl reconfigure

![gitlab-yum](http://wx3.sinaimg.cn/mw690/79b5b049gy1g2b96wk9stj20nx0iwq6i.jpg)

### 2.4 重启gitlab
`gitlab-ctl restart`

使用管理员账户登录后可以看到gitlab的版本号已经从11.2.1升到了11.2.3。
![gitlab-update](http://ws4.sinaimg.cn/mw690/79b5b049gy1g2b9791ywpj20bd0evgmd.jpg)
