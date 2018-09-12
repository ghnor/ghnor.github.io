Hexo本身有插件能实现自动部署的功能（或者说一部分）。

之前是怎么做的呢。

在Github上建一个仓库，博客原文件放在hexo分支，生成的静态网页放在master分支。每次添加了文章，都得先把原文件push到hexo分支，再部署博客到master分支。

但我这个人特别善变，又喜新厌旧。哪天要是不想用Hexo了咋办，我那些文章上有很多hexo特定的内容，比如：hexo新建文章时自动添加的头部信息，包括title、date；分割文章用到的`<!--more-->`标签。

**我想要的：**

1. 保留文章的原始内容

2. 可选择性的发布文章

## 最开始的时候

为了解决问题1，就新建了一个仓库存储原始文章，每次呢文章就先传到这个新仓库上，然后复制文章的内容到博客的仓库。

试了两次，觉得很傻，放弃！

再然后想到Github提供Webhook的功能，但是需要一台服务器。一是我本身不懂服务端，二是就为了这破事儿没必要专门弄台服务器。

没钱技术又差，放弃！

## Travis CI

既然不弄自己的服务器，就找找看有没有免费的持续化部署方案。果然发现有很多CI服务，就选择了Travis CI。

一是免费，二是搜索的时候发现Travis CI出现的频次比较高。用的人多，出现问题好解决。

那么，现在的流程应该就变成：

1. push文章到原始文章仓库，自动通知Travis CI

2. Travis CI把博客仓库clone下来，把原始文章处理一下然后push到博客仓库的hexo分支

3. Travis CI再生成Hexo博客静态网页内容，并部署到博客仓库的master分支

齐活！

## 动手

首先对上面做一点小的优化，原始文章其实不需要单独存放一个仓库，作为博客仓库的一个分支就可以。那么现在博客仓库的分支情况就变成了：原始文章放在source分支；博客原始文件放在hexo分支；博客静态网页文件放在master分支。

博客的第一次部署，还是采用手动的方式。Hexo博客怎么部署本文不会详述。

稍微介绍下Travis CI，因为我发现现在网上的一些文章在拿到GitHub Personal Access Token之后，是手动加密Token然后再保存在.travis.yml文件中。其实没有必要这么做，在Travis CI网站上可以直接配置在项目中，让Teavis CI自动帮你加密。

### 配置Travis CI

打开[Github](https://github.com)，找到用户设置页面，如下：

![](https://shanghai-1252949174.cos.ap-shanghai.myqcloud.com/20180907/20180907174235.png)

点击Generate new token，会让你输入Token的描述，还有选择权限。感觉有一百万个权限，英语文盲，看不懂，所以除了删除仓库（delete_repo）的权限不勾，其他都勾上。再点击Generate token，记得把Token保存起来，因为它只有这次能看到，以后就看不到了。

打开[Travis CI](https://travis-ci.org/)。

这里说下，Travis CI有两个地址，一个以org结尾，是免费的，另一个以com结尾，是收费的。

![](https://shanghai-1252949174.cos.ap-shanghai.myqcloud.com/20180907/20180907175042.png)

如果是第一次用，应该是空的，点击那个+号，就能看到下面这样的，是你Github账户下全部的仓库。把你想自动部署的仓库选上。

![](https://shanghai-1252949174.cos.ap-shanghai.myqcloud.com/20180907/20180907175335.png)

点击More options -> Settings，把上面复制的GitHub Personal Access Token添加进去，名字随意。

![](https://shanghai-1252949174.cos.ap-shanghai.myqcloud.com/20180907/20180907175445.png)

![](https://shanghai-1252949174.cos.ap-shanghai.myqcloud.com/20180907/20180907175901.png)

### 原始文章分支

首先在source分支下添加.travis.yml和.settings.yml。

.settings.yml就是配置文件，在这里声明过的文章，才会被添加到Hexo博客上。配置原始文章的路径、标题以及一些Hexo生成博客文章需要的内容。

当然，这里不是Hexo也不是Travis CI的功能，我写了一个nodejs脚本来完成这部分的工作。

代码很简单，就是把配置文件的内容和文章的内容拼凑在一次，代码就不贴了。

```yml
-
  moreLoc: 7
  path: Android/源码分析 - Router @chenenyu.md
  title: 源码分析 - Router @chenenyu
  date: 2018-08-29 14:58:00
  updated: 2018-08-29 14:58:00
  tags:
    - Android
  categories:
    - Android
```

.travis.yml就是利用Travis CI完成上面说的第二步。把ghnor.github.io仓库clone下来，根据上面的配置文件，生成文章。并推送到博客仓库的hexo分支。

```yml
language: node_js

node_js: stable

before_script:
  - export TZ='Asia/Shanghai'  # 更改时区
  - npm install yamljs
  - git clone https://${GITHUB_REF}  # GITHUB_REF是最下面配置的仓库地址
  - cd ghnor.github.io
  - git checkout -t origin/hexo
  - cd ..

script:
  - node generate.js

after_script:
  - cd ghnor.github.io
  - git config user.name "ghnor"  # 修改name
  - git config user.email "ghnor.gmail.com"  # 修改email
  - git add .
  - git commit -m "Travis CI Auto Build at `date +"%Y-%m-%d %H:%M"`"  # 提交记录包含时间 跟上面更改时区配合
  - git push --force --quiet "https://${GITHUB_TOKEN}@${GITHUB_REF}"  # GITHUB_TOKEN是在Travis中配置环境变量的名称

env:
  global:
    - GITHUB_REF: github.com/ghnor/ghnor.github.io.git
```

### 博客分支

这里就没花里胡哨的东西了，纯粹使用的Travis CI的功能，直接贴.travis.yml。

```yml
language: node_js

node_js: stable

before_install:
  - export TZ='Asia/Shanghai'  # 更改时区

install:
  - npm install -g hexo-cli  # 安装hexo及插件
  - npm install

script:
  - hexo clean  # 清除
  - hexo generate  # 生成

after_script:
  - git clone https://${GITHUB_REF} _public  # GITHUB_REF
  - cd _public
  - git checkout -t origin/master
  - cd ../
  - mv _public/.git/ public/
  - cd public  # 这一步之前的操作是为了保留master分支的提交记录，不然每次git init的话只有1条commit
  - git config user.name "ghnor"  # 修改name
  - git config user.email "ghnor.me@gmail.com"  # 修改email
  - git add .
  - git commit -m "Travis CI Auto Build at `date +"%Y-%m-%d %H:%M"`"  # 提交记录包含时间 跟上面更改时区配合
  - git push --force --quiet "https://${GITHUB_TOKEN}@${GITHUB_REF}" master:master  # GITHUB_TOKEN是在Travis中配置环境变量的名称
  - git push --force --quiet "https://ghnor:${CODING_TOKEN}@${CODING_REF}" master:master

env:
  global:
    - GITHUB_REF: github.com/ghnor/ghnor.github.io.git
    - CODING_REF: git.coding.net/ghnor/ghnor.coding.me.git
```