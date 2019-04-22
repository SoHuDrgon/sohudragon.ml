---
title: "SVN项目迁移Gitlab"
date: 2019-04-22T11:37:20+08:00
tags: [“Gitlab”]
categories: [“Gitlab”]
draft: false
---

## SVN项目迁移到Gitlab教程


### 一.准备工作
	• 本地安装Git，下载地址：https://git-scm.com/downloads，安装即可
	• 我们的Gitlab地址：http://*********，没有账号的自行注册
	• 找到C:\Users\用户名\.ssh文件夹，复制其中id_rsa.pub，粘贴到Rrofile Settings->SSH Keys的Key中，Title随便填写
	• 本地磁盘任意位置建立空文件夹，作为Git本地仓库


### 二.SVN迁移到Gitlab 
#### 进入到需要迁移项目SVN地址，查看提交记录，即show log中对应的Author，本地建立.txt文件

编辑文件格式如下：
```
zhangsan = zhang<11111111@qq.com>
lisi = li<22222222@qq.com>
....
wangwu = wang<33333333@qq.com>
```

***
第一个参数zhangsan是SVN对应Author的名字，  
第二个参数zhang是zhangsan对应Gitlab的名字，<此用户Gitlab邮箱>


我的实例：
```
#cat auther.txt 
nixiaodong = nixiaodong<nixd@xxx.com>
zhongyajie = zhongyajie<zhongyj@xxx.com>
shijiajia = shijiajia<shijj@xxx.com>
xuehao = xuehao<xuehao@xxx.com>
gaowei = gaowei<gaowei@xxx.com>
baijiangbo = baijiangbo<baijb@xxx.com>
donggang = donggang<donggang@xxx.com>
yanchong = yanchong<yanchong@xxx.com>
lichuntian = lichuntian<lict@xxx.com>
yanlei = yanlei<yanlei@xxx.com>
lixiaodong= lixd<lixd@xxx.com>
```


SVN提交log中有几名不同的Author，这里对应需要有几条记录，缺少的话会出现迁移失败，重复的无需添加，
>打开Git安装目录下的git-bash.exe，切换到Git本地仓库路径下，执行下面命令完成Git初始化
>>git config --global user.name "用户名"      与gitlab相同  
>>git config --global user.name "邮箱"         与gitlab相同

在Git本地仓库目录下建立需要迁移项目同名的空文件夹（同名为了方便识别）

git-bash命令行中执行执行：  

`git svn clone yoursvnaddress --authors-file=D:\\users.txt D:\\Test\\KEntity`

第一处参数是svn地址，需要换成需要迁移的项目地址，
第二个参数是上文建立的.txt文件路径，
第三个参数为上文建立的Git本地仓库下新建的需要迁移项目同名的空文件的路径，

项目成功down到本地，在远程仓库中建立相同名称的项目（private权限），成功后复制项目ssh地址，例如：yourgitlabaddress  
git-bash中执行
*git remote add origin yourgitlabaddress*，把本地库与远程仓库关联  
git-bash中执行
*git push -u origin master*  

## 三.Gitlab权限管理
[gitlab权限控制详情文档](https://docs.gitlab.com/ee/user/permissions.html)

通过Gitlab上点击项目的Members可控制权限，通过Add new user to filename
共包含四种权限：

**Guest**       
**Reporter**      
**Developer**       
**Master**  


**权限由小到大**  
>Guest（暂时无用）  
>Reporter可以提供给测试人员 ，可以下载，不能提交    
>Developer可以下载和提交项目，但是只能提交到非保护的分支（master主干分支是受保护的），再由root权限或者Master进行合并（建议分配给一般开发者使用）   
>Master既可以下载也可以进行提交，对受保护的分支合并（建议分配给确定的一到两个人） 

## 四.实际操作过程记录
### 1.登录Gitlab，创建项目
![gitlab-project](http://ws3.sinaimg.cn/mw690/79b5b049gy1g2beplsmwaj20zh0k3ady.jpg)

### 2.初始页面有命令行指引

命令行指引

Git 全局设置  
```
git config --global user.name "Administrator"
git config --global user.email "admin@example.com"
```
创建一个新存储库
```
git clone https://gitlab.xxx.com/k8stest/portalconsole.git
cd portalconsole
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```
现有的文件夹
```
cd existing_folder
git init
git remote add origin https://gitlab.xxx.com/k8stest/portalconsole.git
git add .
git commit -m "Initial commit"
git push -u origin master
```
现有的Git存储库
```
cd existing_repo
git remote rename origin old-origin
git remote add origin https://gitlab.xxxx.com/k8stest/portalconsole.git
git push -u origin --all
git push -u origin --tags
```

### 3.下载svn项目
我这里内网和公网都配置了，我使用内网ip拉代码了，它会自动生成文件夹存储：  
`git svn clone  svn://192.168.11.230/project/dev2/portalConsole --authors-file=auther.txt portalConsole`

拉取代码完成后，我们就使用命令行指引的 *现有的文件夹* 那段话：  
```
git init
git remote add origin http://192.168.11.240/chinagpay/portalConsole.git
git add .
git commit -m "Initial commit"
git push -u origin master
```
操作完成后，gitlab代码库上面就有我们的代码了。
