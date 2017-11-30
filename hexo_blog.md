<header class="site-header">

<div class="header-inside">

<div class="logo">![]()</div>

<nav class="navbar">

<div class="collapse">

*   [首页](/.)
*   [文章列表](/archives)

</div>

</nav>

<div class="button-wrap"><button class="menu-toggle">Primary Menu</button></div>

</div>

</header>

<div class="content-area">

<div class="post">

<div class="container">

<article>

<div class="post-header">

# 教你10分钟搭建属于自己的Blog

2017-02-09

<div class="tags-links">[其他](/tags/其他/)</div>

</div>

<div class="entry-content">

码农都很健忘，为了方便记忆，很多人都会把工作中遇到的问题或者学到的知识，写到博客里，以方便自己查阅，同时也可以帮助他人。
写博客的方式，基本可以分为三种方式：第一，直接去csdn、博客园等博客平台写；第二，自己买服务器和域名，自己搭建；第三，使用gitPages等平台工具搭建。每种方式都可以,每一种方式也都有自己的优缺点，今天来简单介绍一下第三种方式：使用gitPages和Hexo来搭建。

## [](#gitPages "gitPages")gitPages

如果你对编程有所了解，就一定听说过github。它号称程序员的Facebook，有着极高的人气，许多重要的项目都托管在上面。
简单说，它是一个具有版本管理功能的代码仓库，每个项目都有一个主页，列出项目的源文件。
但是对于一个新手来说，看到一大堆源码，只会让人头晕脑涨，不知何处入手。他希望看到的是，一个简明易懂的网页，说明每一步应该怎么做。因此，github就设计了Pages功能，允许用户自定义项目首页，用来替代默认的源码列表。所以，github Pages可以被认为是用户编写的、托管在github上的静态网页。现在，很多人开始用它来写博客。
gitPages用的十分广泛，很多知名项目的帮助文档，都是拿gitPages来做的。例如：[React](https://facebook.github.io/react/)

### [](#创建一个repository "创建一个repository")创建一个repository

打开自己的github,创建一个名为“username.github.io”的项目，**_注意：username必须是自己的github的用户名_**
![](/img/pages_create_repo.png)

### [](#Clone-the-repository "Clone the repository")Clone the repository

克隆项目

```
git clone https://github.com/username/username.github.io

```

### [](#Hello-World "Hello World")Hello World

进入到你克隆下来的项目目录里，添加一个html文件

```
cd username.github.io
echo "Hello World" > index.html

```

### [](#Push-it "Push it")Push it

把它推到github上，Add, commit, and push your changes:

```
git add --all
git commit -m "Initial commit"
git push -u origin master

```

### [](#访问 "访问")访问

最后访问你的website **_[http://username.github.io](http://username.github.io)_**

## [](#Hexo "Hexo")Hexo

好了，现在你相当于有了一个免费的域名外加一个只能写静态页面的server。现在你只需要更新一下你的html，git push到你对应的gitHub的repository上，外网就可以访问到你最新的内容。有人会说：“假如每次写博客都手动写一遍前端代码，太麻烦了，写完了还得自己发布到github,有没有一种工具可以帮我们干这件事呢？” 答案是有的，例如[**_Hexo_**](https://hexo.io/zh-cn/)。

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。Hexo的作用，就是提供一个你写博客的小工具，在你写完之后，可以把你使用Markdown语法写在md文件上的内容制作成带有主题的静态页面，并发布到github（或者其他git托管平台）上。这样，你就可以仅关注写博客本身，而无需做一些开发性的工作。

Hexo的使用，也很简单：

### [](#安装 "安装")安装

安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否安装了Node.js和Git。
如果电脑上已经装完了Node.js和Git,则：

```
npm install -g hexo-cli

```

### [](#建站 "建站")建站

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```
$ hexo init <folder>
$ cd <folder>
$ npm install 

```

新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml       网站的配置信息。您可以在此配置大部分的参数。
├── package.json      应用程序的信息。
├── scaffolds         scaffolds模版文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。 Hexo的模板是指在新建的markdown文件中默认填充的内容。
├── source            source资源文件夹。是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。
|   ├── _drafts       Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。
|   └── _posts
└── themes           主题文件夹。Hexo 会根据主题来生成静态页面。

```

输入一下指令

```
hexo generate  (此指令可以简化为:hexo g)
hexo server （此指令可以简化为:hexo s）

```

启动服务器。默认情况下，访问网址为： [http://localhost:4000/](http://localhost:4000/)

### [](#部署 "部署")部署

推送生成的html代码到github，以便外网访问。

到blog的根目录下。打开_config.yml配置文件。翻到最后一行，编辑deploy相关的信息。内容如下:

```
deploy:
  type: git
  repo: https://github.com/i6448038/i6448038.github.io.git   你的github仓库地址
  branch: master

```

配置结束以后。输入以下指令，把代码自动部署到github。

```
hexo deploy （此指令可以简化为：hexo d）

```

如果你输入以上指令后没有反应，那么你应该：

```
npm install hexo-deployer-git --save

```

成功后，你就可以访问自己的gitPages了，[https://username.github.io](https://username.github.io)

### [](#写博客 "写博客")写博客

打开gitPages后发现都是程序默认生成的东西，现在需要自己写自己博客的内容了。

```
hexo new "文章题目"  (此指令可以简化:hexo n "文章题目")

```

其中”RyuGou”为文章标题，执行命令 hexo n “RyuGou” 后，会在项目 \Hexo\source_posts 中生成 RyuGou.md文件，用编辑器打开编写即可。
编辑内容使用的是MarkDown语法。有关MarkDown的内容再次就不做赘述了。
详情请看：[http://wowubuntu.com/markdown/basic.html](http://wowubuntu.com/markdown/basic.html)
或者 [http://wowubuntu.com/markdown/index.html#philosophy](http://wowubuntu.com/markdown/index.html#philosophy)
或者 [http://www.markdown.cn/](http://www.markdown.cn/)
关于MarkDown的编辑器，也多种多样，Atom、sublime、Mou、甚至vim也可以。

写完内容后，如果想改变当前博客的风格，可以去Hexo Themes的官网选择自己喜欢的风格。 [https://hexo.io/themes/](https://hexo.io/themes/)
打开网址：
![](/img/website.png)
点击喜欢的内容，会自动打开到该theme的gitHub仓库。
![](/img/theme.png)
Aloha对应的gitHub仓库：
![](/img/github.png)
然后你需要做如下操作:

```
$ cd $YOUR_BLOG_ROOT_DIR 
$ git clone https://github.com/henryhuang/hexo-theme-aloha.git themes/aloha

```

然后更改当前文件夹下的_config.yml，把该文件中theme的值改为aloha。除此之外，如果你想更改该风格的站点菜单栏字段等文本内容，可以去theme文件夹下的aloha子文件夹下的__config.yml文件中修改。
然后执行

```
$ hexo g
$ hexo s

```

浏览器打开 [http://localhost:4000](http://localhost:4000) 看看风格是否改变

</div>

</article>

</div>

<div class="nav-links">

<div class="nav-previous">[<span class="meta-arraw meta-arraw-left"></span>Older Posts](/2017/03/05/让vim炫酷起来/)</div>

</div>

</div>

</div>

<div class="footer-widgets">

<div class="row inside-wrapper">

<div class="col-1-3">

<aside>

# 关于本站

<div class="custom-widget-content">

*   <a href="">RyuGou的个人网站</a>

</div>

</aside>

</div>

<div class="col-1-3">

<aside>

# Contact

<div class="widget-text">[github](https://github.com/i6448038) [mail](376832293@qq.com)</div>

</aside>

</div>

<div class="col-1-3">

<aside>

# Search

<div class="widget-text">

<form onsubmit="return appDaily.submitSearch('')">

<input type="text" placeholder="search..." id="homeSearchInput">

</form>

</div>

</aside>

</div>

</div>

</div>

<footer class="site-info">

<span>RyuGou的博客 © 2017</span> <span class="split">|</span> <span>Powered by [RyuGou](https://hexo.io/)</span>

</footer>
