---
title: 使用Hexo与GitHub搭建独立域名博客
abbrlink: 3eeb
date: 2020-03-01 13:00
summary: '搭建博客是有两个必须要条件：博客框架、托管平台，框架我们采用Hexo，而平台大部分人会选择GitHub。属于比较适合代码托管平台，相比于Gitee来说，可以绑定域名，但是在访问速度方面较差。'
categories: 环境部署
tags: 
- Hexo
- GitHub
- 博客
- 域名
---

## 环境要求

- Git
- Nodejs
- 安装Hexo及代码提交托管需要以上两个条件

---

### 安装 Git

#### Windows平台下安装

Windows平台下安装相关要简单，下载安装包一路下一步便是。因此不在多叙述。[Git官网下载](https://git-scm.com/)  [淘宝镜像下载](https://npm.taobao.org/mirrors/git-for-windows/v2.25.0.windows.1/Git-2.25.0-32-bit.exe)

#### CentOS平台安装

在CentOS平台安装最好是先更新为国内Yum源，并更新系统软件等包为最新。这样速度较快，支持较好。

``` bash
$ sudo yum install git
```

#### Ubuntu平台安装

在Ubuntu平台安装与在CentOS相差不大

``` bash
$ sudo apt install git
```

#### Mac平台安装

在Mac平台安装既可以通过GUI的方式来安装也可以通过Command的方式来安装，使用Homebrew进行安装：

``` bash
$ sudo brew install git
```

如果没有安装HomBrew， [可以参考](https://brew.sh/index_zh-cn)

### Git 配置命令

``` bash
#使用前配置全局信息
git config --global user.name '你的名字'
git config --global user.email '你的邮箱'
#查看是否已配置成功
git config --list --global
#在你需要使用Git的文件夹下运行
git init                        #初始化
git add .                        #添加变更文件到暂存区
git status                        #查看变更信息
git commit -m '你需要写的信息'        #提交到工作区
git remote add origin https://github.com/用户名/仓库名.git    #添加Github远程连接
git push origin                 #上传到GitHub
git push origin -f                #强制上传到GitHub
```

#### Git代理相关

查看当前的代理设置

``` bash
git config --global http.proxy
git config --global https.proxy
```

设置当前代理

``` bash
git config --global http.proxy 'http://127.0.0.1:1080'
git config --global https.proxy 'http://127.0.0.1:1080'
```

或

``` bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

删除代理

``` bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

其它Git用法请参考[文档](https://git-scm.com/book/zh/v2)

### 安装 Node

#### Windows平台安装

Windows平台安装Node较简单，不再多叙述。 [Node下载](http://nodejs.cn/download/)

安装完后输入 node -v 查询版本信息

#### CentOS平台

``` bash
$ sudo yum install nodejs npm
```

源码安装,先下载 Node源码

``` bash
$ sudo tar -xvf node-v10.16.0-linux-x64.tar.xz
$ sudo mv node-v10.16.0 /home/blue/applications
$ ln -s /home/blue/applications/node /home/blue/applications/node
$ sudo chmod -R 755 /home/blue/applications/node-v10.16.0
$ sudo chmod -R 755 /home/blue/applications/node
$ cd /home/blue/applications/node
$ sudo ./configure
$ sudo make && make install
$ sudo node --version
v10.16.0
```

#### Ubnutu平台安装

``` bash
$ sudo apt-get install nodejs npm
```

#### Mac平台安装

``` bash
$ sudo brew install node npm
```

#### 设置国内NPM源

``` bash
# 永久设置全局淘宝镜像源
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
# 临时修改镜像源
npm --registry=https://registry.npm.taobao.org
# 永久设置为淘宝镜像源
npm config set registry https://registry.npm.taobao.org
# 查看npm的配置
npm config list
```

#### 安装 Yarn (非必须）

``` bash
npm install -g yarn
# 配置yarn淘宝源
yarn config set registry 'https://registry.npm.taobao.org'
# 设置 npm 缓存
npm config set prefix "/home/blue/applications/cache/node/prefix"
npm config set cache "/home/blue/applications/cache/node/cache"
```

还需要将/home/blue/applications/cache/node/prefix添加到PATH环境变量

## 博客搭建

### 安装 Hexo

``` bash
npm install -g hexo   #安装hexo
npm install           #安装博客需要的依赖文件
# 配置国内淘宝 cnpm ( 使用npm较慢时可改用)
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
$ sudo cnpm install -g hexo
```

### Hexo 初始化

创建一个hexo仓库文件夹，进入文件夹初始化hexo

```bash
$ cd /home/hexo
$ hexo init
INFO Cloning hexo-starter .....
```

初始化完成后，在hexo目录下生成相关文件

hexo 目录结构

- _config.yml          配置文件
- _public            生成的静态文件，这个目录最终会发布到服务器
- _scaffolds         通用模板
- _source         保存编写的markdown文件
- drafts            草稿文件
- themes             博客主题
- node_modules     类库

### 安装博客主题

在hexo目录中运行

```bash
git clone https://github.com/blinkfox/hexo-theme-matery themes/matery
# 启用主题
$ vi _config.yml
# 修改文件中的 theme
theme: matery
```

克隆完成后，在/Hexo/themes目录下，可以看到 landscape和matery 两个文件夹。
我们所要使用的主题都是放在这个目录下，Hexo默认使用的是landscape主题，NexT主题用的比较多且更多样化，我们这一步克隆了next主题，接下来会使用next主题进行演示。
想获取更多主题，可在网站：[此处](https://hexo.io/themes/)选择自己喜欢的主题，按照此步的步骤clone下来。

### hexo目录中_confit.yml文件配置

```bash
# Site
# 博客名称
title: aiotlab
# 副标题 
subtitle: aiotlab blog
# 个人简介
description: 这是 aiotlab blog
keywords: aiotlab,lag,aiotlab
# 博主
author: Jeremy Peng
# 语言
language: zh-CN
# 时区
timezone: Asia/Shanghai
url    :网址    
root :网站根目录    
permalink: 文章的永久链接格式 :year/:month/:day/:title/
permalink_defaults:    永久链接中各部分的默认值    
pretty_urls: 改写 permalink 的值来美化 URL    
pretty_urls.trailing_index: 是否在永久链接中保留尾部的 index.html，设置为 false时去除
pretty_urls.trailing_html: 是否在永久链接中保留尾部的 .html, 设置为 false 时去除
# 目录(基本不需改)
source_dir        资源文件夹，这个文件夹用来存放内容
public_dir        公共文件夹，这个文件夹用于存放chang生成的站点文件
tag_dir            标签文件夹
archive_dir        归档文件夹
category_dir    分类文件夹
code_dir        Include code 文件夹，source_dir 下的子目录
i18n_dir        国际化（i18n）文件夹
skip_render        跳过指定文件的渲染。(常用于跳过GitHub的README.md渲染)
# 分页
per_page        每页显示的文章量 (0 关闭分页功能,默认10)
pagination_dir    分页目录
# 主题,当前主题名称
theme: matery
# 发布
deploy:
  type: git
  repo: 仓库
  branch: 分支
```

#### 新建分类categories页

```bash
hexo new page "categories"
```

编辑文件 /source/categories/index.md，内容如下：

```bash
---
title: categories
date: 2020-03-01
type: "categories"
layout: "categories"
---
```

#### 新建标签 tags 页

```bash
hexo new page "tags"
```

编辑文件 /source/tags/index.md，内容如下：

```bash
---
title: tags
date: 2020-03-01 18:23:38
type: "tags"
layout: "tags"
---
```

#### 新建关于我 about 页

```bash
hexo new page "about"
```

编辑文件 /source/about/index.md，内容如下：

```bash
---
title: about
date: 2020-03-01 17:25:30
type: "about"
layout: "about"
---
```

#### 新建友情连接 friends 页（可选的）

```bash
hexo new page "friends"
```

编辑文件 /source/friends/index.md，内容如下：

```bash
---
title: friends
date: 2020-03-01 21:25:30
type: "friends"
layout: "friends"
---
```

同时，在你的博客 source 目录下新建 _data 目录，在 _data 目录中新建 friends.json 文件，文件内容如下所示：

```txt
[{
    "avatar": "https://cdn.jsdelivr.net/gh/jeremysvn/aiotlab/medias/avatar.jpg",
    "name": "AiotLab",
    "introduction": "这是我的博客",
    "url": "https://aiotlab.info/",
    "title": "前去学习"
}, {
    "avatar": "https://s2.ax1x.com/2020/02/13/1q6iAs.th.png",
    "name": "AiotLab",
    "introduction": "这是我的其他博客",
    "url": "http://aiotlab.cc/",
    "title": "前去查看"
}]
```

####  新建留言contact页

```bash
hexo new page "contact"
```

编辑文件 /source/contact/infex.md 内容如下：

```bash
---
title: contact
date: 2020-03-01 21:25:30
type: "contact"
layout: "contact"
---
```

### 发布测试

#### 本地发布测试

```bash
hexo clean && hexo g && hexo s
...
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

访问 http://localhost:4000/ 即可看到博客效果

## 博客部署

### 创建Github仓库

访问 https://github.com/ ，申请注册账号，并创建一个仓库

### 配置_config.yml

``` bash
deploy:
  type: git
  repository: git@github.com:帐号/仓库名.git
  branch: master
```

### 配置 ssh key

#### 创建 ssh key

``` bash
ssh-keygen -t rsa -C “aiotlab@126.com”
```

连续三个或四个回车，生成密钥，最后得到了两个文件：id_rsa和id_rsa.pub

##### 添加密钥到ssh-agent

```bash
eval "$(ssh-agent -s)"
```

##### 添加生成的SSH key到ssh-agent

``` bash
ssh-add ~/.ssh/id_rsa
```

登录Github，点击头像下的settings，选择右边的ssh and GPG keys 添加ssh
新建一个new ssh key,将生成的id_rsa.pub文件里内容粘贴上面就行啦

测试是不否添加成功

``` bash
ssh -T git@github.com
```

如果看到后面显示的是你的git用户名，说明添加成功。

配置Deployment，在其文件夹中，找到_config.yml文件，修改deploy中的repo值（在末尾）
repo值是你的github项目中右边Clone or download可以看到

## 发布项目

安装自动部署发布工具

```bash
npm install hexo-deployer-git --save
```

发布命令

```bash
hexo clean && hexo g && hexo d
```

发布时会提示输入github帐号和密码（未添加ssh key），提示发布完成

### 设置CNAME

在 hexo 项目下，source 文件夹下面创建 CNAME 文件（没有后缀名的），在里面写上购买的域名。比如：

``` bash
baize.cc
```

在 github 上面，打开 username.github.io 项目的（Settings）设置，然后在 GitHub Pages的 Custom domain设置里填上购买的域名。

打开你添加的域名，是否发布成功。
