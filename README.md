# 笔记

此仓库为 [https://sandy1890.github.io](https://sandy1890.github.io) 网站的内容源，该仓库的内容用于生成博客网站

### 初次运行

* 需要node环境支持
* 安装hexo及初始化项目

```bash
$ npm install -g hexo-cli
$ npm install
```

* 使用NexT主题

仓库里没有存放Next主题相关代码,如果在一个新环境使用，在第一次发布之前，需要先到根目录执行下载命令

```bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```


###　写作及发布

* 将写作的 md 文件放到　source/_posts　目录下
* scaffolds 目录下是模版文件
* 本地生成
``` bash
# hexo原生方式
$ hexo generate

# npm 命令
$ npm run gen
```

* 本地预览,运行命令后浏览器中打开命令行输出的地址，如:http://localhost:4000/

```bash
$ npm run server
> hexo s

INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

```

* 写作完成后可以使用以下 `` npm 命令``或 ``deploy.sh`` 脚本进行发布

```bash 
# deploy.sh脚本
$ sh deplsy.sh

# node命令
$ npm run deploy
```
## 官方关于写作的文档　

### 写作
你可以执行下列命令来创建一篇新文章。

``` bash
$ hexo new [layout] <title>
```

您可以在命令中指定文章的布局（layout），默认为 `post`，可以通过修改 `_config.yml` 中的 `default_layout` 参数来指定默认布局。

### 布局（Layout）

Hexo 有三种默认布局：`post`、`page` 和 `draft`，它们分别对应不同的路径，而您自定义的其他布局和 `post` 相同，都将储存到 `source/_posts` 文件夹。

布局 | 路径
--- | ---
`post` | `source/_posts`
`page` | `source`
`draft` | `source/_drafts`

{% note tip 不要处理我的文章 %}
如果你不想你的文章被处理，你可以将 Front-Matter 中的`layout:` 设为 `false` 。
{% endnote %}

### 文件名称

Hexo 默认以标题做为文件名称，但您可编辑 `new_post_name` 参数来改变默认的文件名称，举例来说，设为 `:year-:month-:day-:title.md` 可让您更方便的通过日期来管理文章。

变量 | 描述
--- | ---
`:title` | 标题（小写，空格将会被替换为短杠）
`:year` | 建立的年份，比如， `2015`
`:month` | 建立的月份（有前导零），比如， `04`
`:i_month` | 建立的月份（无前导零），比如， `4`
`:day` | 建立的日期（有前导零），比如， `07`
`:i_day` | 建立的日期（无前导零），比如， `7`

### 草稿

刚刚提到了 Hexo 的一种特殊布局：`draft`，这种布局在建立时会被保存到 `source/_drafts` 文件夹，您可通过 `publish` 命令将草稿移动到 `source/_posts` 文件夹，该命令的使用方式与 `new` 十分类似，您也可在命令中指定 `layout` 来指定布局。

``` bash
$ hexo publish [layout] <title>
```

草稿默认不会显示在页面中，您可在执行时加上 `--draft` 参数，或是把 `render_drafts` 参数设为 `true` 来预览草稿。

### 模版（Scaffold）

在新建文章时，Hexo 会根据 `scaffolds` 文件夹内相对应的文件来建立文件，例如：

``` bash
$ hexo new photo "My Gallery"
```

在执行这行指令时，Hexo 会尝试在 `scaffolds` 文件夹中寻找 `photo.md`，并根据其内容建立文章，以下是您可以在模版中使用的变量：

变量 | 描述
--- | ---
`layout` | 布局
`title` | 标题
`date` | 文件建立日期


## GitHub Pages　

###　使用GitHub搭建免费的个人博客 

在这里我使用了两个仓库，一个用于存放写作源码，另外一个用于发布博客

具体操作步骤为:

* 在github创建一个用于存放heox项目的仓库，仓库名随意，用于将写作内容提交到这个仓库保存

* 在github创建另一个名为 ``{你的用户名}.github.io`` （将``{你的用户名}``替换为你自己的账号，注意，这个仓库名称必须如此）新的仓库,这个仓库用于接收hexo提交上来的博客页面,github会自动将此仓库的master分支作为以 https://{你的用户名}.github.io 域名的空间 [参看:https://help.github.com/articles/user-organization-and-project-pages/](https://help.github.com/articles/user-organization-and-project-pages/)

* 配置_config.yml中 deploy 项 详细操作查看[官方文档](https://hexo.io/zh-cn/docs/deployment.html)

```ini
deploy:
    type: git
    repo: https://github.com/sandy1890/sandy1890.github.io.git
    branch: master
```


### GitHub pages 限制
We recommend GitHub Pages users follow these limits:

主页空间容量1GB
GitHub Pages source repositories have a recommended limit of 1GB .
Published GitHub Pages sites have a 1GB recommended limit.

每月100GB流量或100,000请求
GitHub Pages sites have a recommended bandwidth limit of 100GB or 100,000 requests per month.

限制每小时10次构建
GitHub Pages sites have a recommended limit of 10 builds per hour.

GitHub Pages官方说明 [https://help.github.com/articles/what-is-github-pages](https://help.github.com/articles/what-is-github-pages)

### 相关链接
* [hexo配置文档](https://hexo.io/zh-cn/docs/index.html)
* [NexT主题文档](http://theme-next.iissnan.com/getting-started.html)
