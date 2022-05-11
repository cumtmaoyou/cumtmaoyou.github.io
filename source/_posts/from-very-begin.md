---
title: 从一开始
abbrlink: 13d30382
date: 2022-05-11 15:17:01
categories:
 - 记录生活
 - Hexo
tags:
 - 博客
---
## 源起
我在自己服务器上搭建的Typecho博客被备案这种事情折腾得头疼，终于忍不住放弃了；又见识了各种各样通过GithubPages创建的博客，终于忍不住想要试试Hexo。
## 折腾
零零散散大约花了两个上午的时间，算是折腾好了。这边也简单记一下折腾的步骤。
### 环境
首先跟着官方文档来，按部就班地装npm，接着`nmp install -g hexo-cli`，简单两步环境就好了。接下来是通过`hexo init`建站。其实这个时候已经可以通过`hexo s`来启动站点在本地查看了，但是当时折腾的时候先折腾的是部署到Github Pages，所以这里也按照这个顺序来记录一下。
### 部署到Github
要部署到GithubPages，需要一个Github的仓库，可以参考[GitHub创建仓库](https://docs.github.com/cn/get-started/quickstart/create-a-repo)。仓库的名称一定要是`{你的账号}.github.io`才能使用[{你的账号}.github.io](https://cumtmaoyou.github.io)这个域名来访问。然后官方文档上说通过TravisCI来自动部署，但是这个东西已经很老了，我大概看了下TravisCI的步骤，要使用就需要选择一个Plan，即使选了免费的Plan也需要你填写信用卡账户，到这步我相信90%的人都可以放弃了。这里我选择了Github的Actions了作为替代，相当好用。
1. 配置部署密钥
```
# ~: ssh-keygen -f github-deploy-key
```
生成github-deploy-key和github-deploy-key.pub文件
- 将github-deploy-key文件的内容添加到仓库的`Settings -> Secrets -> Add a new secret`中的Value，Name可以自己取，这边用HEXO_DEPLOY_PRI
- 将github-deploy-key.pub文件的内容添加到仓库的`Settings -> Deploy Keys -> Add Deploy Key`中的Key，Title可以自己取，这边用HEXO_DEPLOY_PUB，添加的时候记得勾选上Allow write access
2. 在hexo站点根目录下创建`.github/workflows/deploy.xml`，xml的大致内容可参考[我的deploy.yml](https://github.com/cumtmaoyou/cumtmaoyou.github.io/blob/master/.github/workflows/deploy.yml)
```
# 简单解释一下
name: auto_ci # Action的名字
on.push.branches: master # 表示master有推送就自动执行Action
env: # 记录的是一些环境变量
jobs: # Action执行的任务列表
steps: # 任务步骤
  - name: Checkout # 步骤名字
    uses: actions/checkout@v3 # 执行checkout操作，check的就是当前仓库

  - name: Checkout theme repo
    uses: actions/checkout@v3
    with:
      repository: ${{ env.THEME_REPO }} # check主题仓库，地址是env.THEME_REPO中填写的仓库
      ref: ${{ env.THEME_BRANCH }} # 仓库分支名称
      path: themes/next # 本地目录
    
  # 这一步是执行npm install
  - name: Use Node.js ${{ matrix.node_version }}
    uses: actions/setup-node@v3
    with:
      node-version: ${{ matrix.node_version }}
```
3. 修改Hexo的配置文件_config.yml
```
deploy:
  type: 'git'
  repo: git@github.com:cumtmaoyou/cumtmaoyou.github.io.git
  branch: autoci
```
4. 在`Settings -> Pages -> Source`中选择autoci分支

这个时候如果推送修改到Github，那么就可以自动执行Actions了，并且会自动发布到autoci分支
### 使用自己的域名
在Source文件夹下创建CNAME文件，填入自己的域名即可
### 修改主题
主题的折腾永无止境，可以到[Hexo主题列表](https://hexo.io/themes/)选择自己喜欢的主题。我在折腾了三、四个主题后还是选择和[NexT](https://github.com/next-theme/hexo-theme-next)，简单大气。当然也少不了个性话的定制了。
在仓库的Source文件夹下创建_data文件夹，同时创建主题配置里提到的文件
```
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  head: source/_data/head.njk
  header: source/_data/header.njk
  sidebar: source/_data/sidebar.njk
  postMeta: source/_data/post-meta.njk
  postBodyEnd: source/_data/post-body-end.njk
  footer: source/_data/footer.njk
  bodyEnd: source/_data/body-end.njk
  variable: source/_data/variables.styl
  mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```
接下来就根据需要修改styles.styl就可以定制css了

### post使用标签和分类
```
在post的开头---之间加入
categories:
 - 记录生活
tags:
 - Green
```

## 后记
这是一个全新的博客，也是全新的起点，希望自己能坚持下来，不要半途而废
