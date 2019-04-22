---
title: "CentOS 6 安装Gitlab"
date: 2019-04-22T10:15:14+08:00
tag: “Gitlab”
category: “Gitlab”
draft: false
---


## Gtilab Omnibus安装教程
	
	Gitlab官网：www.gitlab.com
	官网安装说明：https://docs.gitlab.com/ce/install/requirements.html
	官方安装教程：https://about.gitlab.com/install/#centos-6
	一键安装脚本：https://packages.gitlab.com/gitlab/gitlab-ce/install#bash-rpm

### 官方建议
我们强烈建议安装Omnibus软件包，因为它安装更快，更易于升级，并且包含增强其他方法所没有的可靠性的功能。我们还强烈建议至少4GB的可用内存来运行GitLab。

### 编辑YUM源
使用清华大学 TUNA 镜像源 打开网址将内容复制到gitlab-ce.repo文件中，编辑路径
```
cat >> /etc/yum.repos.d/gitlab-ce.repo << EOF
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
EOF
```

### 更新本地 YUM 缓存
`sudo yum makecache`

### 安装 GitLab 社区版

`sudo yum install gitlab-ce` 			#(自动安装最新版)

`sudo yum install gitlab-ce-x.x.x`    	#安装指定版本

### 安装完成更改Gitlab配置
```
 vim /etc/gitlab/gitlab.rb
 # 找到 external_url 'http://192.168.11.240'
 # 修改成你的地址
```

### 配置并启动GitLab
```
# 打开`/etc/gitlab/gitlab.rb`,
# 将`external_url = 'http://git.example.com'`修改为自己的IP地址：`http://xxx.xx.xxx.xx`，
# 然后执行下面的命令，对GitLab进行编译。
sudo gitlab-ctl reconfigure
```

### 登录GitLab
初始登录会提示更改管理员密码
Username: root 
Password: xxxxxxx

### GitLab头像无法正常显示
原因：gravatar被墙

解决办法：
```
编辑 /etc/gitlab/gitlab.rb，将
# gitlab_rails['gravatar_plain_url'] = 'http://gravatar.duoshuo.com/avatar/%{hash}?s=%{size}&d=identicon'
修改为：
gitlab_rails['gravatar_plain_url'] = 'http://gravatar.duoshuo.com/avatar/%{hash}?s=%{size}&d=identicon'
```
然后在命令行执行：
`sudo gitlab-ctl reconfigure` 
`sudo gitlab-rake cache:clear RAILS_ENV=production`

### nginx配置
解决 80 端口被占用
```
upstream gitlab {
     server ip:8081 ;
}
server {
    #侦听的80端口
    listen       80;
    server_name  gitlab.xxx.com;
    location / {
        proxy_pass   http://gitlab;    #在这里设置一个代理，和upstream的名字一样
        #以下是一些反向代理的配置可删除
        proxy_redirect             off;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header           Host $host;
        proxy_set_header           X-Real-IP $remote_addr;
        proxy_set_header           X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size       10m; #允许客户端请求的最大单文件字节数
        client_body_buffer_size    128k; #缓冲区代理缓冲用户端请求的最大字节数
        proxy_connect_timeout      300; #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout         300; #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout         300; #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size          4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers              4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
        proxy_busy_buffers_size    64k; #高负荷下缓冲大小（proxy_buffers*2）
        proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传
    }
}
```

#### 检查配置
```
/opt/gitlab/embedded/sbin/nginx -tc /var/opt/gitlab/nginx/conf/nginx.conf 
nginx: the configuration file /var/opt/gitlab/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /var/opt/gitlab/nginx/conf/nginx.conf test is successful
```

#### nginx 重新加载配置
`/opt/gitlab/embedded/sbin/nginx -s reload`


## 运维
### 启动所有 gitlab 组件：
`sudo gitlab-ctl start`

#### 停止所有 gitlab 组件：
`sudo gitlab-ctl stop`

#### 重启所有 gitlab 组件：
`sudo gitlab-ctl restart`

#### 查看服务状态
`sudo gitlab-ctl status`

#### 启动服务
`sudo gitlab-ctl reconfigure`

#### 修改默认的配置文件
`sudo vim /etc/gitlab/gitlab.rb`

#### 查看版本
`sudo cat /opt/gitlab/embedded/service/gitlab-rails/VERSION`

```
# echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
# sysctl -p
# echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

#### 检查gitlab
`gitlab-rake gitlab:check SANITIZE=true --trace`

# 查看日志
`sudo gitlab-ctl tail`

## 备份恢复
### Gitlab 创建备份
使用Gitlab一键安装包安装Gitlab非常简单, 同样的备份恢复与迁移也非常简单,用一条命令即可创建完整的Gitlab备份:
`gitlab-rake gitlab:backup:create`

以上命令将在/var/opt/gitlab/backups目录下创建一个名称类似为xxxxxxxx_gitlab_backup.tar的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的xxxxxx是备份创建的时间戳。

### Gitlab 修改备份文件默认目录
修改/etc/gitlab/gitlab.rb来修改默认存放备份文件的目录:
`gitlab_rails['backup_path'] = '/mnt/backups'`
修改后使用gitlab-ctl reconfigure命令重载配置文件。

### 定时备份
```
0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

## gitlab恢复
首先进入备份 gitlab 的目录，这个目录是配置文件中的 gitlab_rails['backup_path'] ，默认为 /var/opt/gitlab/backups 。
然后停止 unicorn 和 sidekiq ，保证数据库没有新的连接，不会有写数据情况。
```
# 停止相关数据连接服务
# ok: down: unicorn: 0s, normally up
gitlab-ctl stop unicorn  
# ok: down: sidekiq: 0s, normally up
gitlab-ctl stop sidekiq

# 从xxxxx编号备份中恢复
# 然后恢复数据，1406691018为备份文件的时间戳
gitlab-rake gitlab:backup:restore BACKUP=xxxxxx

# 启动Gitlab
sudo gitlab-ctl start
```

## 汉化
其实官方已经汉化过了，只是下面的汉化是增强而已，可以不用汉化的。
https://gitlab.com/xhang/gitlab


## 邮箱配置
```
# cat /etc/gitlab/gitlab.rb
### Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'example@email.com'
gitlab_rails['gitlab_email_display_name'] = 'somename you set'
gitlab_rails['gitlab_email_reply_to'] = 'noreply'


### GitLab email server settings
###! Docs: https://docs.gitlab.com/omnibus/settings/smtp.html
###! **Use smtp instead of sendmail/postfix.**

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qiye.163.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "zabbix@163.com"
gitlab_rails['smtp_password'] = "xxxxxxxxxx"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```


## gitlab https配置

1.创建证书存储目录，并把证书放入：
```
# pwd
/etc/gitlab/ssl
# ll
total 12
-rw-r--r-- 1 root root 1634 Jan 21 14:04 ca.crt
-rw-r--r-- 1 root root 2272 Jan 21 14:03 xxx.com.crt
-rw-r--r-- 1 root root 1679 Jan 21 14:04 xxx.com.key
```

2.配置gitlab的配置文件gitlab.rb
```
################################################################################
## GitLab NGINX
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html
################################################################################

nginx['enable'] = true
nginx['client_max_body_size'] = '250m'
nginx['redirect_http_to_https'] = false #如果需要http和https共存就不跳转了。
# nginx['redirect_http_to_https_port'] = 80

##! Most root CA's are included by default
nginx['ssl_client_certificate'] = "/etc/gitlab/ssl/ca.crt"

##! enable/disable 2-way SSL client authentication
# nginx['ssl_verify_client'] = "off"

##! if ssl_verify_client on, verification depth in the client certificates chain
# nginx['ssl_verify_depth'] = "1"

nginx['ssl_certificate'] = "/etc/gitlab/ssl/xxx.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/xxx.com.key"
nginx['ssl_ciphers'] = "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"
nginx['ssl_prefer_server_ciphers'] = "on"
```

3.配置完成后重配置和重启gitlab 服务
  Run configuration wizard (Chef Solo Setup)
  `sudo gitlab-ctl reconfigure`
  Restart Services
  `sudo gitlab-ctl restart`


4.配置http和https共存
https://github.com/sameersbn/docker-gitlab/issues/716

将http的配置文件保存一份，/var/opt/gitlab/nginx/conf/gitlab-http.conf，将其中的server的那段文件复制进新的https的配置文件里面即可：
```
server {
  listen *:80;


  server_name 192.168.11.240;
  server_tokens off; ## Don't show the nginx version number, a security best practice

  ## Increase this if you want to upload large attachments
  ## Or if you want to accept large git objects over http
  client_max_body_size 0;


  ## Real IP Module Config
  ## http://nginx.org/en/docs/http/ngx_http_realip_module.html

  ## HSTS Config
  ## https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/
  add_header Strict-Transport-Security "max-age=31536000";

  ## Individual nginx logs for this GitLab vhost
  access_log  /var/log/gitlab/nginx/gitlab_access.log gitlab_access;
  error_log   /var/log/gitlab/nginx/gitlab_error.log;

  if ($http_host = "") {
    set $http_host_with_default "192.168.11.192";
  }

  if ($http_host != "") {
    set $http_host_with_default $http_host;
  }

  gzip on;
  gzip_static on;
  gzip_comp_level 2;
  gzip_http_version 1.1;
  gzip_vary on;
  gzip_disable "msie6";
  gzip_min_length 10240;
  gzip_proxied no-cache no-store private expired auth;
  gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/json application/xml application/rss+xml;

  ## https://github.com/gitlabhq/gitlabhq/issues/694
  ## Some requests take more than 30 seconds.
  proxy_read_timeout      3600;
  proxy_connect_timeout   300;
  proxy_redirect          off;
  proxy_http_version 1.1;

  proxy_set_header Host $http_host_with_default;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection $connection_upgrade;
  proxy_set_header X-Forwarded-Proto http;

  location ~ (\.git/gitlab-lfs/objects|\.git/info/lfs/objects/batch$) {
    proxy_cache off;
    proxy_pass http://gitlab-workhorse;
    proxy_request_buffering off;
  }

  location / {
    proxy_cache off;
    proxy_pass  http://gitlab-workhorse;
  }

  location /assets {
    proxy_cache gitlab;
    proxy_pass  http://gitlab-workhorse;
  }

  error_page 404 /404.html;
  error_page 500 /500.html;
  error_page 502 /502.html;
  location ~ ^/(404|500|502)(-custom)?\.html$ {
    root /opt/gitlab/embedded/service/gitlab-rails/public;
    internal;
  }

  
}
```

最后，restart（千万不要reconfig，没有修改gitlab.rb，不要reconfig）。就可以看到http和https都可以了。

配置完成后不要重新配置，只需要重启即可：
`gitlab-ctl restart`


