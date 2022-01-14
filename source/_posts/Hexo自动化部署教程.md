---
title: Hexo自动化部署教程
date: 2022-01-14 22:23:15
tags: CI/CD Blog Tutorial
---

此篇文章献给平安，读完这篇文章，你将会知道如何部署自己的博客到公网上并且自动化这一过程。

1. 安装 hexo

![image-20220114223522332](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114223522332-7e218618f05355ab9271174ddb7b4f832c0e631247b7380da4ed4212de1b8041.png)

成功后你打开[http://localhost/4000/](http://localhost/4000/)应该就能看到博客已经成功运行到本地了。

当你想添加一篇新文章时，运行`hexo new 文章名称` hexo就会在对应`source/_posts`目录新建一个`文章名称.md`的markdown文件。另外运行`hexo build`可以将我们写的文章编译成静态的网页文件到`public`文件夹里。

2. Gihub新建Repo

   这次我们借助github提供的pages服务来部署我们的博客到公网。这次需要到github上新建一个repo. Repo的名称请用你的`<账号名>.github.io`命名。类似我的github账户名是`narutiohinata`。所以我新建的Repo名称就是 `narutohinata.github.io`。

   ![image-20220114225124151](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114225124151-5107e783f5c1139f34ef28cb34953d67945baf52516d3788d2dcc0690b0c0bd8.png)

   完成后，将你本地博客项目push到刚刚新建的Repo里。这需要你在你的博客项目初始化git

   ```bash
   # 初始化git
   git init
   
   #这一步是为了和github的默认主分支名称统一
   git branch -m main
   
   # 添加 Github 仓库到本地
   git remote add origin https://github.com/username/username.github.io.git
   # 将所有文件添加到 git
   git add .
   
   # 添加 commit
   git commit -m "initial"
   
   # 将本地的文件推送到 Github 上的 main 分支
   git push -u origin main
   
   ```

   成功之后你应该在你的Repo里看到内容。

3. 配置Travis CI

首先你需要到[https://www.travis-ci.com/](https://www.travis-ci.com/)去注册你的Travis CI的账号(使用你的github账号注册)。成功后登录进去，然后点击头像进入`Settings`页面。这里你应该可以看到你的github账号对应的Repos，找到你博客的Repo，点击最右侧的`Settings`按钮。



![image-20220114231602743](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114231602743-2d1b4ce78c8efb71135bf97ad40f6b119c017ff5e5a8c6a61c898c06f10b2295.png)

在进入的Settings页面，找到Enviroment Variables模块，在这里我们需要去`Github`申请一个`Personal access token`。

回到github,点击右上角头像进入Settings页面。在侧边栏找到`Developer settings`选项，点击它进入`Developer settings`页面。然后找到`Personal access tokens`选项。在这里新建一个token, 确保勾选中repo，expiration过期时间我这里推荐No expiration。不然的话你可能需要定期更新token。

![image-20220114232635544](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114232635544-d33579adacb4917bd8488e0551bfc5b003a0710367fd2faba0253dcc2a44e427.png)

完成后点击最下方的Generate Token按钮，将得到的`token`保存到上一步Travis CI里的Enviroment Variable里，为了保持和我的一致，你也把环境变量名称设为GH_TOKEN吧。

![image-20220114232904230](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114232904230-9eef5d9f73fdae4625d1fb9863f52532cadca7f626a0bd1be740688844ed3709.png)

到这一步，Travis CI应该算配置ok了，下一步需要回到我们项目目录给它添加.tarvis.yml文件。



4. 配置.travis.yml

我们在本地博客项目根目录添加`.travis.yml`文件。然后贴上下面的配置

```yaml
os: linux
language: node_js
install:
  - yarn
node_js:
  - 13 # use nodejs v13 LTS-
after_success:
  - git config --global user.name "<your nickname>"
  - git config --global user.email "<your email>"
cache: npm
branches:
  only:
    - main # build master branch only
script:
  - yarn build # generate static files
deploy:
  strategy: git
  provider: pages
  skip_cleanup: true
  token: $GH_TOKEN
  keep_history: true
  on:
    branch: main
  local_dir: public
```

然后将这个新添加的.travis.yml文件提交到版本库，同时push到github上去(稍微解释下这个配置文件，这个文件会告诉travis ci服务我需要一个linux同时拥有nodejs版本为13的部署环境，我需要安装yarn. 在成功之后配置git config, 克隆main分支的内容，成功后运行`yarn build`生成静态网页文件,然后会部署到名为`gh-pages`的分支)。这时如果你看到你的github的博客仓库的git commit值之前出现了一个小图标，看起来像是加载什么东西。那么恭喜你，证明travis ci已经在工作了。过了一段时间如果看到那个图表变成了绿色的小对钩，证明构建部署工作完成。检查下你的分支是不是有了gh-pages分支。

![image-20220114234446257](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114234446257-784a2160ca888723488ae7d04b19d97544dfd89c712f395907110b69c0c3bb72.png)

5. 配置Github Pages服务

如果上面步骤一切顺利，我们就可以开始配置pages服务了。在你的博客仓库页面，点击上方的Settings选项。在进入的设置页面选择左侧栏的Pages选项卡。具体可参考github这篇文章[configuring-a-publishing-source-for-your-github-pages-site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

![image-20220114234739230](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/image-20220114234739230-5c020d678d43d29dd8a4c4cd3efeca4d7a5144ed1d83232a43a627a2cdd921b2.png)
