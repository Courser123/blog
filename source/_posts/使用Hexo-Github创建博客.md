---
title: 使用Hexo + Github创建博客
date: 2019-05-14 15:23:33
tags:
---

## Hexo

### 什么是 Hexo？

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 安装

#### 安装前提

- [Node.js](https://nodejs.org/en/) (Should be at least nodejs 6.9)
- [Git](https://git-scm.com/)

如果你的电脑中已经安装上述必备程序,接下来只需要使用 npm 即可完成 Hexo 的安装。

`$ npm install -g hexo-cli`

如果你的电脑中尚未安装所需要的程序，请根据以下安装指示完成安装。

> Mac 用户
您在编译时可能会遇到问题，请先到 App Store 安装 Xcode，Xcode 完成后，启动并进入 Preferences -> Download -> Command Line Tools -> Install 安装命令行工具。

#### 安装Git

- Windows：下载并安装 [git](https://git-scm.com/download/win).
- Mac：使用 [Homebrew](https://brew.sh/), [MacPorts](https://www.macports.org/) ：`brew install git`;或下载 [安装程序](https://sourceforge.net/projects/git-osx-installer/) 安装。
- Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

#### 安装Node.js

安装 Node.js 的最佳方式是使用 [nvm](https://github.com/nvm-sh/nvm)。

cURL:

``` bash
$ curl https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

Wget:

``` bash
$ wget -qO- https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

安装完成后，重启终端并执行下列命令即可安装 Node.js。

```bash
$ nvm install stable
```

或者也可以下载 [安装程序](https://nodejs.org/en/) 来安装。

#### 安装 Hexo
所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```bash
$ npm install -g hexo-cli
```

### 建站

安装 Hexo 完成后，执行下列命令建站，Hexo 将会在指定文件夹中新建所需要的文件。

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

#### 本地预览

此时已经可以本地预览Blog页面了,执行下列命令

```bash
$ hexo server # 开启本地服务器
```

终端会输出本地url,默认是[http://localhost:4000](http://localhost:4000),打开可预览本地Blog页面

### 与GitHub建立关联

#### 新建GitHub仓库(如果已经创建,请跳到下一步)

新建一个GitHub仓库,名称为: `用户名.github.io.git # 将用户名替换为你自己的github用户名`

#### 生成 ssh key(如果已经生成,请跳到下一步)

```bash
$ ssh-keygen -t rsa -C "your_email@example.com" # 注意将`your_email`替换成之前注册github帐号时的邮箱
```

然后进入 `~/.ssh/id_rsa.pub` 路径下,打开`id_rsa.pub`,将内容复制到剪切板

登录[GitHub](https://github.com/), 点击Settings –> SSH and GPG keys –> New SSH key,将SSH key添加到GitHub账号中

#### 配置_config.yml(在Hexo生成的根目录下)

打开_config.yml,找到deployment模块,将改模块替换为如下格式(**格式要一模一样,包括空格和换行之后的位置**):
```bash
deploy: 
  type: git
  repo: https://github.com/用户名/用户名.github.io.git
  branch: master
 ```
`_config.yml`其他配置参考官方文档 [配置_config.yml](https://hexo.io/zh-cn/docs/configuration)

### 将博客推至远端

#### generate

```bash
$ hexo generate
```
生成静态文件。

选项	| 描述
----| ---
-d, --deploy | 文件生成后立即部署网站

#### deploy

```bash
$ hexo deploy
```
部署网站。

参数	| 描述
----| ---
-g, --generate | 部署之前预先生成静态文件

#### 其他指令

参见 [Hexo指令](https://hexo.io/zh-cn/docs/commands)

### 打开博客页面

浏览器输入:[https://用户名.github.io](https://用户名.github.io),打开博客页面,博客创建成功!

由于Hexo使用MarkDown解析,附上链接[MarkDown语法说明](http://wow.kuapp.com/markdown/)
