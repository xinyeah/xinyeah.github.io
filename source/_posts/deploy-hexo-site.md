---
title: 搭建Hexo + Github Pages + Travis CI个人站点的详细教程
date: 2020-06-07 21:27:46
tags: Hexo; git submodule; Next; Travis CI; Github Pages
categories: 搭建站点
description: 本文详细介绍了如何快速的搭建Hexo + Github Pages + Travis CI个人站点。以Next主题为例，介绍了在项目中添加了作为git submodule的主题后，如何正确的部署站点和发表文章。以及如何使用Travis CI将Hexo项目自动部署到Github Pages。
---

## 技术栈选择： Github Pages + Hexo + Travis CI

首要原因：没钱。这是一套**免费**的组合拳。

在众多站点选择中，最终选择了Github Pages。主要还是因为熟悉Github的版本控制，以及Github对其他平台很好的集成。官方推荐的静态站点生成器(static site generator)是[Jekyll](https://help.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll)。还可以在项目仓库Settings页面中的Github Pages部分，选择Jekyll的theme。

Hexo也是一款静态站点生成框架(static site generator)，基于Node.js 。通过Hexo可以使用Markdown来写文章，不用太关注排版和格式。而且Hexo比较成熟，有很多稳定的好看的主题。

Travis CI 是持续集成(continuous integration)的平台，可以监控repo具体分支上的代码变动，自动触发build和test。帮助实现频繁的merge小段代码的Best practice。有了自动部署，就可以不受开发平台限制，不需要搭建环境也可以发布文章。

## 安装环境

1. 安装并配置github

2. 安装Node.js 和 npm

   npm(Node Package Manager), 是用来开发和分享Javascript代码的工具。在这里https://nodejs.org/en/download/下载Node.js的最新版本，就包含了NPM。

   打开Command Prompt，验证安装成功了Node.js和NPM。

   ``````
   $ node –v
   v12.17.0
   
   $ npm –v
   6.14.4
   ``````

3. 安装Hexo

   ```
   $ sudo npm install -g hexo-cli
   
   $ hexo -v
   hexo-cli: 3.1.0
   os: Windows_NT 10.0.18363 win32 x64
   node: 12.17.0
   ...
   ```

4. 安装Hexo-deployer-git

   ```
   npm install hexo-deployer-git --save
   ```

   

##  初始化Hexo+Github Pages项目

1. Github上创建一个<YourName>.github.io为名的公开的代码库。其中Yourname应该跟你的Github用户名保持一致。代码库Settings中查看Github Pages相关设置，你就拥有了自己的站点：https://<YourName>.github.io。对于个人站点，只能将master分支设置为发布来源。

   ![image-20200617193336642](/images/image-20200617193336642.png)

2. 点击Clone, 复制代码库的URL。

![image-20200617181657118](/images/image-20200617181657118.png)

2. 初始化<YourName>.github.io为Hexo项目

   ```
   $ git clone https://github.com/<YourName>/<YourName>.github.io.git
   
   $ cd <YourName>.github.io
   
   $ hexo init <YourName>.github.io
   INFO  Copying data to ~/***/<YourName>.github.io
   INFO  You are almost done! Don't forget to run 'npm install' before you start blogging with Hexo!
   
   $ npm install
   ```

3. 初始化后的目录如下：

   > .
   > ├── _config.yml   #站点的配置文件
   > ├── package.json   #应用的基本信息和依赖应用
   > ├── scaffolds   #模板文件夹。新建文章时候，默认填充的内容模板。
   > ├── source   #markdown和html文件会被解析存放在public文件夹中
   > |   ├── _drafts   #新建的draft会保存在这里
   > |   └── _posts   #新建post的时候会保存在这里
   > └── themes   #主题文件夹，根据主题来生成静态页面

4. 根据[文档](https://hexo.io/docs/configuration)，修改_config.yml文件中关于站点的配置信息。

5. 执行以下命令， 验证效果

   ```
   $ hexo server
   INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
   ```

## 添加博客主题

1. Fork [hexo-theme-next](https://github.com/sugartxy/hexo-theme-next) 项目到自己的仓库.

2. 运行以下命令将 Fork 出来的仓库 pull 到本地子模块

   ```
   cd <YourName>.github.io
   git submodule add https://github.com/<YourName>/hexo-theme-next.git themes/next
   git checkout master
   ```

   运行该命令后会在项目根目录生成 `.gitmodules` 文件，文件内容如下：

   ```
   [submodule "themes/next"]
       path = themes/next
       url = https://github.com/sugartxy/hexo-theme-next
   ```

3. 对主题进行个性化配置后，先要 check in子模块，在 theme/next 目录下依次执行：

   ```
   cd theme/next
   git add .
   git commit -m "update config file"
   git push origin master
   ```

4. 切换到项目根目录，打开站点配置文件(<YourName>.github.io/_config.yml)，修改theme字段, 使得主题修改生效。

   ```
   theme: next
   ```

5. 执行以下命令， 验证效果

   ```
   $ hexo server
   INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
   ```

6. 在项目根目录下，将代码check in到项目仓库下：

   ```
   cd <YourName>.github.io
   git add .
   git commit -m "add submodule"
   git push origin master
   ```

## 生成博客并部署

1. 执行以下命令生成新的博客

   ```
   $ hexo new post <title>
   INFO  Created: ~/<YourName>/<YourName>.github.io/source/_posts/<title>.md
   ```

   将博客内容写在新创建的markdown文件里。

2. 如果themes/next路径下的内容做了改变，在themes/next路径下，将更改的代码check in到刚刚Fork的repo中。

3. 在YourName.github.io项目路径下，将更改的代码check in到YourName.github.io repo 的master分支.

4. 在本地部署。使用后续Travis CI配置后，可以省略此步骤。

   4.1 修改配置文件`_config.yml`中关于部署的字段

   ```
   deploy:
     type: git
     repository: https://git@github.com/<YourName>/<YourName>.github.io.git
     branch: master
   ```

   4.2  执行以下命令部署站点，当执行 `hexo deploy` 时，Hexo 会将 `public` 目录中的文件和目录推送至 `_config.yml` 中指定的远端仓库和分支中，并且**完全覆盖**该分支下的已有内容。

   ```
   $ hexo clean # 清除缓存文件（db.json）和已经生成的静态文件（public）
   $ hexo generate  # 生成静态文件
   $ hexo deploy  # 部署网站
   ```

##  使用Travis CI自动化部署

Travis CI对于开源的Repository是免费的，只需要拥有Github账户和至少一个项目，在项目中增加.travis.yml文件，就可以使用Travis CI。[Hexo文档](https://hexo.io/zh-cn/docs/github-pages)中详细说明了如何使用Travis CI将Hexo自动部署到Github Pages。只需要做如下修改:

1. 修改.travis.yml文件

   ```
   sudo: required
   language: node_js
   node_js:
     - 10 # use nodejs v10 LTS
   
   branches:
     only:
       - master # build master branch only
   
   
   # Start: Build Lifecycle
   install:
     - npm install -g hexo-cli
     - npm install
     - npm install hexo-deployer-git --save
     # 设置git提交名，邮箱
     - git config user.name "<YourName>"
     - git config user.email "<YourEmail>"
     # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
     - sed -i "s/gh_token/${GH_TOKEN}/g" ./_config.yml
   
   script:
     - hexo clean
     - hexo generate # generate static files
     
   after_success: # 只有前面步骤成功了才会触发
     - hexo deploy
     
   # End: Build LifeCycle
   ```

   

2. 修改_config.yml文件

   ```
   deploy:
     type: git
     # 下方的gh_token会被.travis.yml中sed命令替换
     repo: https://gh_token@github.com/<YourName>/<YourName>.github.io.git
     branch: master
   ```

这样每一次更新博客，只需要check in Markdown文件到master 分支，就会自动部署。在Travis CI网站中可以看到部署的状态。

![image-20200617204126319](/images/image-20200617204126319.png)

## 参考文献

Next中文教程：https://theme-next.iissnan.com/getting-started.html#description-setting

Hexo中文教程：https://hexo.io/zh-cn/docs/

Github Pages中文教程：https://help.github.com/cn/github/working-with-github-pages

Travis官方文档：https://docs.travis-ci.com/user/tutorial/