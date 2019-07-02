---
title: "Hugo Install"
date: 2019-04-17T15:22:41+08:00
tags: [“Hugo”]
categories: [“Hugo”]
draft: false
---

## 安装Hugo
本文在macOS High Sierra环境下写成。Windows平台可参考[官方文档](https://www.gohugo.org/)做相应修改。

如果Mac上已经装上了Homebrew，在命令行中直接敲下面的命令安装：

`brew install hugo`

如果Mac上没有安装Homebrew，可以考虑先装Homebrew再按上述步骤安装Hugo；也可以在Hugo官网直接下载使用可运行文件。
至此Hugo就安装完毕了。So easy.

### 在Hugo中写文章
在硬盘中选取合适的存储路径，然后命令行中使用如下指令生成网页本地文档：
`hugo new site personal-site`

```
hugo new site sohudrgon.ml
Congratulations! Your new Hugo site is created in /Users/guogang/Projects/sohudrgon.ml.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

由此可得到如下文件目录：
```
personal-site
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

常用目录用处如下
```
| 子目录名称 | 功能 |
| ------------ | ---------------------------------------------------------------------- |
| archetypes | 新文章默认模板 |
| config.toml | Hugo配置文档 |
| content | 存放所有Markdown格式的文章 |
| layouts | 存放自定义的view，可为空 |
| static | 存放图像、CNAME、css、js等资源，发布后该目录下所有资源将处于网页根目录 |
| themes | 存放下载的主题 |
```

使用下面的命令生成新的文章草稿：
`hugo new posts/first-post.md`

在content目录中会自动以archetypes/default.md为模板在content/posts目录下生成一篇名为first-post.md的文章草稿：
```
---
title: "First Post"
date: 2017-12-27T23:15:53-05:00
draft: true
---
```

我们可以加一个标题在下面并去掉标记为草稿的这一行：draft: true
```
---
title: "First Post"
date: 2017-12-27T23:15:53-05:00
---
## Hello world
```

然后随便下载一个主题并加载到config.toml文件中：
```
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
# Edit your config.toml configuration file
# and add the Ananke theme.
echo 'theme = "ananke"' >> config.toml
```
```
git submodule add https://github.com/vaga/hugo-theme-m10c.git themes/m10c
echo 'theme = "m10c"' >> config.toml
```
现在使用如下命令建立本地服务器：
`hugo server`

并在浏览器中输入网址http://localhost:1313/就可以在浏览器中查看网页效果了：
![localhostsite](http://wx4.sinaimg.cn/mw690/79b5b049gy1g25qnevi0hj21180m0404.jpg)


如果觉得没有问题了便可以使用如下命令：
`hugo`

如此一来网页便生成在默认的public子目录中了。

## 发布并托管到Github

上传到Github之前，先在Github中添加一个空白repository，注意不要添加如README，.gitignore等文档。由此得到Github中该repository的网址：https://github.com/guogang1990/sohudrgon.ml.git
![repository](http://ws3.sinaimg.cn/mw690/79b5b049gy1g25qmowit5j20k60hdmz9.jpg)

复制该网址后，在网站本地文档根目录中初始化git：
```
git init
git commit -m "first commit"
git remote add origin https://github.com/guogang1990/sohudrgon.ml.git
git push -u origin master
```

至此所有源文档就都push到Github上了。然而此时Github对待这些源文档跟其他任何普通的repository中的代码并没有任何不同，并不会将public子目录中的网页托管在Github Pages上。
参见Hugo官方文档，可以选择以下两种方式让Github Pages加载我们想要托管的/public子目录中的网页：
	1. 配置Hugo将网页生成在名为/docs的子目录中，然后直接push到master branch
	2. 仍然使用默认的/public子目录存储网页，再单独建立一个gh-pages branch

### 使用/docs发布到master branch
第一种方案的好处在于一次push即可将源文档和对应生成的网页文档都发布到Github，操作非常简单。所需要的仅是在config.toml中添加如下一行配置，使得生成的网页默认保存在/docs子目录下：
publishDir = docs

自此运行hugo命令后生成的网页文件将保存在/docs子目录下。将所有文档push到Github的master branch，进入Github对应repository的Settings标签菜单，在GitHub Pages选项的Source栏选择master branch /docs folder:
![docs](http://wx2.sinaimg.cn/mw690/79b5b049gy1g25qltk6cvj20l00iv76p.jpg)


等待片刻即可访问http://your_name.github.io看到之前用Hugo生成的网页了。
![hugosite](http://ws1.sinaimg.cn/mw690/79b5b049gy1g25qle7u5ej20dl0b5t9k.jpg)

### 发布到gh-pages branch
如果希望单独控制源文档和生成的网页文档的版本历史，可以使用单独建立一个gh-pages branch的方法托管到Github Pages——先将/public子目录添加到.gitignore中，让master branch忽略其更新，然后在本地和Github端添加一个名为gh-pages的branch：
```
//忽略public子目录
echo "public" >> .gitignore
//初始化gh-pages branch
git checkout --orphan gh-pages
git reset --hard
git commit --allow-empty -m "Initializing gh-pages branch"
git push origin gh-pages
git checkout master
```

为了提高每次发布的效率，可以将下述命令存在脚本中，每次只需要运行该脚本即可将gh-pages branch中的文章发布到Github的repo中：
```
#!/bin/sh
if [[ $(git status -s) ]]
then
    echo "The working directory is dirty. Please commit any pending changes."
    exit 1;
fi
echo "Deleting old publication"
rm -rf public
mkdir public
rm -rf .git/worktrees/public/
echo "Checking out gh-pages branch into public"
git worktree add -B gh-pages public origin/gh-pages
echo "Removing existing files"
rm -rf public/*
echo "Generating site"
hugo
echo "Updating gh-pages branch"
cd public && git add --all && git commit -m "Publishing to gh-pages (publish.sh)"
echo "Push to origin"
git push origin gh-pages
```

最后将master branch中的源文档和gh-pages branch中的网页文档分别push到Github repo中，进入Settings标签菜单，选择Github Pages项中的Source栏，点gh-pages branch选项：
![gh-pages](http://ws4.sinaimg.cn/mw690/79b5b049gy1g25qkfk3amj20l40abaeg.jpg)

同样等待片刻，即可访问https://your_name.github.io 看到之前用Hugo生成的网页了。

## 配置个人域名

如果觉得使用your_name.github.io不够酷炫，还可以考虑使用自选的个人域名。好的个人域名自然是需要到对应服务商购买的。常见的域名如.com, .net, .me一般都不免费，但好在非顶级域名的年费其实也不贵。由于我建网的初衷只是自娱自乐，暂时并没有付费购买域名的意向，索性直接去[Freenom](https://my.freenom.com/clientarea.php)找了免费域名来用。

目前Freenom平台提供的免费域名后缀为.tk, .ml,ga,cf,gq等。购买域名很简单，先在[Freenom](https://my.freenom.com/clientarea.php)网站上注册账号，然后查看自己想要的域名的价格并根据提示下单即可。考虑到现在machine leanring这么火，我就选了sohudrgon.ml这个免费域名。

域名买好后还需要设置下域名解析。由于默认使用的是Freenom的DNS服务器，所以需要在Manage Domain菜单中配置域名解析规则。在Manage Freeenom DNS选项中添加如下两条规则：
	1. 添加A记录（即地址记录，用来指定域名的IP地址），主机记录（Name）栏填www，记录值(Target)那栏填Github服务器IP地址（或者your_name.github.io的IP地址）
	2. 添加CNAME记录（用于将一个域名映射到另一个域名），主机记录栏填@，记录值那栏填your_name.github.io

其中，Github服务器IP地址是185.199.110.153，而your_name.github.io的IP地址可以在命令行中使用ping命令得到：
```
#ping guogang.github.io
PING guogang.github.io (185.199.110.153): 56 data bytes
64 bytes from 185.199.110.153: icmp_seq=0 ttl=50 time=31.412 ms
64 bytes from 185.199.110.153: icmp_seq=1 ttl=50 time=30.863 ms
```

最后还需要在personal-site/public子目录中需要添加一个名为CNAME的文档，该文件只包含想要替换的个人域名，对我来说即是sohudrgon.ml（不加http）。由于每次使用hugo命令
在/public子目录生成网页的时候该CNAME文件都会被删除，所以最好将该文件放在personal-site/static子目录中，这样运行hugo后该CNAME文件将自动复制到/public目录中。
等待几小时，在浏览器中访问sohudrgon.ml就可以看到熟悉的个人网站页面了。

## 配置CloudFlare以使用HTTPs
之所以想要使用[CloudFlare](https://dash.cloudflare.com)，是因为上一步当我们配置好个人域名后，由于Github Pages不支持在自定义域名中使用HTTPs协议，所以浏览器中访问sohudrgon.ml使用的是HTTP协议。这造成一个弊端：每回用Chrome打开sohudrgon.ml，浏览器都提示该网页不受信任，如果网页中还有待加载的JavaScript代码,就得单独点浏览器地址栏右侧的load按钮才能正常加载全部页面，非常麻烦。再加上考虑到HTTPs协议比HTTP更快更安全，显然应该想办法解决这个问题。
好在CloudFlare为我们提供了一套方便的解决方案，而且是免费的！

首先点开CloudFlare注册账号，输入前面选好的个人域名sohudrgon.ml，CloudFlare会给我们提供众多服务套餐，选择免费的那个套餐即可:)
此时CloudFlare会给我们提供其DNS服务器的IP，此时需要去Freenom的域名管理页面中更新默认DNS服务商到CloudFlare：
![managedns](http://ws3.sinaimg.cn/mw690/79b5b049gy1g25qjkyco8j20mw0jpwg0.jpg)


更改完DNS服务器就可以设置CloudFlare中的各个选项了。首先在Crypto选项标签下，选择使用Full SSL模式以HTTPs协议加载网页：

![屏幕快照 2019-04-17 下午2.01.02](http://ws4.sinaimg.cn/mw690/79b5b049gy1g25qgn5aoqj20t30f1dhm.jpg)

![usehttps](http://ws1.sinaimg.cn/mw690/79b5b049gy1g25qoyiu4sj20t105oq3b.jpg)  
然后是DNS标签栏配置，跟Freenom设置类似：  
![dnsmanage](http://wx2.sinaimg.cn/mw690/79b5b049gy1g25qp9wl0bj20sy0is76m.jpg)

至此配置完毕。其实CloudFlare中还有很多别的选项，可以根据个人喜好进行相应配置。等待几小时，再次访问sohudrgon.ml，可以发现网页已经在HTTPs协议下加载了。这样以来就再也不用去点那个烦人的load按钮了。

## 后记
至此我的新版博客就迁移完毕了，基本满足我现在写文章各项需求。


