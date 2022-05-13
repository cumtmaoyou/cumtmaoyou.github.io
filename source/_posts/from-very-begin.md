---
title: 从一开始
abbrlink: 13d30382
date: 2022-05-11 15:17:01
categories:
  - 折腾小记
  - Hexo
tags:
  - Hexo
---

## 0x00.源起

我在自己服务器上搭建的 Typecho 博客被备案这种事情折腾得头疼，终于忍不住放弃了；又见识了各种各样通过 Github Pages 创建的博客，终于忍不住想要试试 Hexo。

## 0x01.折腾

零零散散大约花了两个上午的时间，算是折腾好了。这边也简单记一下折腾的步骤。

### 环境

这部分比较简单，跟着官方文档来，安装[nodejs](https://nodejs.org/en)，完成后执行`nmp install -g hexo-cli`，简单两步环境就好了。接下来是创建一个文件夹，在新建的文件夹内通过`hexo init`命令建站。完后就可以通过`hexo s`来启动站点，并通过<a href="#">localhost:4000</a>在本地查看。但是在本地的 Blog 就失去了它的意义，我们需要把它部署到服务器上。这里我们选用 Github Pages 来部署我们的博客。

<!--more-->

### 部署到 Github

1. 创建 Github 仓库
   首先我们创建一个[Github](https://github.com)账号，并[创建一个仓库](https://docs.github.com/cn/get-started/quickstart/create-a-repo)来存储我们的博客。仓库的名称一定要是`{Github账号}.github.io`才能使用[{Github 账号}.github.io](https://cumtmaoyou.github.io)这个域名来访问。官方文档上说可以通过 TravisCI 来自动部署，但是这个东西已经很老了，我大概看了下 TravisCI 的步骤，要使用就需要选择一个 Plan，即使选了免费的 Plan 也需要你填写信用卡账户，到这步我相信 90%的人都可以放弃了。这里我选择了 Github Actions 作为替代。
2. 配置新的 Action 密钥

```
生成公钥和私钥，以下命令将生成"github-deploy-key"和"github-deploy-key.pub"文件
~: ssh-keygen -f github-deploy-key
```

- 通过仓库的`Settings -> Secrets -> Add a new secret`菜单，添加一个新的 Secret，Name 可以自己取，这边用 HEXO_DEPLOY_PRI，Value 使用 github-deploy-key 文件的内容
- 通过仓库的`Settings -> Deploy Keys -> Add Deploy Key`菜单，添加新的 Deploy Key，Title 可以自己取，这边用 HEXO_DEPLOY_PUB，Key 使用 github-deploy-key.pub 文件的内容，添加的时候记得勾选上 Allow write access

3. 添加部署配置
   在 hexo 站点根目录下依次创建`.github`, `workflows`文件夹，并在 workflows 文件夹中创建`deploy.yml`，复制下面内容到 yml

```
name: auto_ci # Action的名字

# 当推送到master分支的时候出发该Action
on:
  push:
    branches:
      - master

# 环境变量设置
env:
  GIT_USER: cumtmaoyou #github账号
  GIT_EMAIL: cumtxiaofeng@live.com #github邮箱
  THEME_REPO: next-theme/hexo-theme-next #当前使用的Hexo的主题repo
  THEME_BRANCH: master # 主题的repo分支名称
  DEPLOY_REPO: cumtmaoyou/cumtmaoyou.github.io #自动部署到该repo
  DEPLOY_BRANCH: autoci #自动部署的repo的分支名称

# 任务列表
jobs:
  build:
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }} #编译的名称 matri.node_version和matrix.os引用的下方的的os和node_version的配置
    runs-on: ubuntu-latest # 表明在ubuntu上编译
    strategy:
      matrix:
        os: [ubuntu-latest] # 系统用最新的ubuntu
        node_version: [16.x] # node用16.x版本

    # 编译步骤
    steps:
      - name: Checkout # 首先是checkout该deploy.yml所在的仓库
        uses: actions/checkout@v3

      - name: Checkout theme repo # checkout主题仓库
        uses: actions/checkout@v3
        with:
          repository: ${{ env.THEME_REPO }}
          ref: ${{ env.THEME_BRANCH }}
          path: themes/next # check到themes/next文件夹下，这个文件夹与第一步所check仓库的根目录的同级

      - name: Checkout deploy repo #checkout部署仓库
        uses: actions/checkout@v3
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
          path: .autoci #check到.autoci文件夹，这个文件夹与第一步所check仓库的根目录的同级

      - name: Use Node.js ${{ matrix.node_version }} # 安装node
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node_version }}

      - name: Configuration environment # 配置自动部署环境
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL
      #          cp .config.next.yml themes/next/_config.yml #这边可以复制一些额外的自定义文件到主题
      - name: Install dependencies # 执行node install
        run: |
          npm install

      - name: Deploy hexo # 执行hexo的deploy命令
        run: |
          npm run deploy

```

4. 修改 Hexo 的配置文件\_config.yml

```
deploy:
  type: 'git'
  repo: {$这里写github的repo地址，如果你启用了ssh，这边要写ssh地址，否则写http地址，简单说就是这个地址要可以正常使用git clone命令}
  branch: {$这里写自动部署到的repo分支名称}
```

5. 设置 Github Pages
   在仓库的`Settings -> Pages -> Source`菜单中选中 autoci 分支

这个时候如果推送本地博客到 Github，那么就可以自动执行 Actions 了，并且会自动发布到 autoci 分支

### 使用自己的域名

在 Source 文件夹下创建 CNAME 文件，填入自己的域名即可

### 修改主题

主题的折腾永无止境，可以到[Hexo 主题列表](https://hexo.io/themes/)选择自己喜欢的主题。我在折腾了三、四个主题后还是选择和[NexT](https://github.com/next-theme/hexo-theme-next)，简单大气。当然也少不了个性话的定制了。
如果需要定制主题的页面，那么：

- 在站点的 Source 文件夹下创建\_data 文件夹
- 在\_data 文件夹下创建以下十个文件
  `head.njk header.njk sidebar.njk post-meta.njk post-body-end.njk footer.njk body-end.njk variables.styl mixins.styl styles.styl`
- 在主题配置里面搜索`custom_file_path`，并删除接下来十行前面的#号，以启用自定以文件

修改 styles.styl，在里面写入你定制的 css

### 添加 gitalk

NexT 主题已经集成了 gitalk，但是需要几个步骤来启用

1. 你需要一个保存评论的仓库，这里我选择仓库就是博客的仓库
2. 创建 OauthApp，在个人 Settings，注意不是仓库的 Settings 里面，选择`Developer settings`，选择`Oauth Apps`，选中`New Oauth App`

```
Application Name: 填入一个有意义的名字
Homepage URL: 这个就写你博客的地址，如(https://{你的github账号.github.io})
Application Description: 自己写一个描述性文字
Authorization callback URL: 这个比较重要，必须得写你博客的地址
```

3. 创建完成后点击 Generate New client secret，并保存生成的 Secret
4. 修改主题配置
   搜索`comments:`，修改 active 为 gitalk
   修改 gitalk 配置

```
gitalk:
  enable: true
  github_id: 仓库的拥有者账号
  repo: 这边写仓库的名字，注意这边不是url，只要写仓库名字，如cumtmaoyou.github.io，写错了就会出现Error Not Found，我在这边卡了很久
  client_id: 这个里写你刚才创建的Oauth App的Client ID
  client_secret: 这个就是刚才生产的Secret
  admin_user: 仓库拥有账号，这里可以有多个，用[]来填，可以写合作者账号
  distraction_free_mode: false # Facebook-like distraction free mode
  language: zh-CN
```

5. 到评论保存仓库的 issue 下创建一个初始评论
6. 到这里就配置完成了，注意每次写一个新的 post 要自己等下 github 初始化一条评论，这里有自动化方法，但是我觉得自己登一下也不是很麻烦

### 关于字体的修改

1. 在主题的配置文件中搜索 Font settings，修改 global 即可，这里 font family 用一个英文的字体
2. 在 style.styl 里用 css 的 font-face 创建一个本地字体，用 css 来精细调整每个 html 标签的字体
3. 如果要用不同于 global 里配置的中文字体，在站点的 source/\_data/variables.styl 里面填入以下文本

```
$font-family-chinese = "Noto Serif SC", "PingFang SC", "Microsoft YaHei";

$font-family-base = $font-family-chinese, sans-serif;
$font-family-base = get_font_family('global'), $font-family-chinese, sans-serif if get_font_family('global');

$font-family-logo = $font-family-base;
$font-family-logo = get_font_family('title'), $font-family-base if get_font_family('title');

$font-family-headings = $font-family-base;
$font-family-headings = get_font_family('headings'), $font-family-base if get_font_family('headings');

$font-family-posts = $font-family-base;
$font-family-posts = get_font_family('posts'), $font-family-base if get_font_family('posts');

$font-family-monospace = consolas, Menlo, monospace, $font-family-chinese;
$font-family-monospace = get_font_family('codes'), consolas, Menlo, monospace, $font-family-chinese if get_font_family('codes');
```

{% note primary %}
暂时折腾这么多了，有空再慢慢玩
{% endnote %}

## 0x02. hexo 的一些使用方法

1. post 使用标签和分类

```
在post的开头---之间加入
categories:
 - 记录生活
tags:
 - Green
```

2. 添加 note

```
{% note default %}
这里面note块的内容
{% endnote %}
```

这里 default 可以替换为`primary success info warning danger`

## 0x03. 后记

这是一个全新的博客，也是全新的起点，希望自己能坚持下来，不要半途而废
