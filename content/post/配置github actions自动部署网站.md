+++
date = '2026-04-01T11:11:56+08:00'
draft = false
title = '基于hugo配置github Actions自动部署网站'
+++

# 配置github Actions自动部署网站


## 分本地和远程两个部分


### 整体的技术架构

**hugo  + github pages + github actions**

**hugo**: 生成静态网站,直接在本地md文件编辑网站内容，使用hugo生成静态网页内容,不用配置任何其他内容
**github pages**: 托管网站,将生成的静态网页内容托管到github pages上，用户可以通过访问github pages的url来查看网站
**github actions**:自动部署网站:当本地代码有变化时，自动触发github actions，部署网站到github pages


### 本地配置

1. 安装hugo


windos 环境下使用powershell安装hugo
```
winget inatall Hugo.hugo.Extended
```

验证版本
```
hugo version
```
![alt text](/images/image.png)


安装git

有多种方式，推荐使用winget安装
```
winget install Git.git
```
验证版本
```
git --version
```
![alt text](/images/image-1.png)
不需要魔法上网

如果上述方式你不安装成功，也可以到中文官方网址下载最新版
```
https://git-scm.cn/install/windows
```



2. 创建hugo项目
```
hugo new site git-blog
cd git-blog
```
3. 配置hugo项目
进入新创建的项目，找到hugo.toml，配置基本信息
![alt text](/images/image-2.png)
配置如下:
```
baseURL = "https://你的用户名.github.io/" # 后面会提及，“你的用户名.github.io”是github pages托管的项目名称，用户名为git账户名称
languageCode = "zh-cn" 
title = "我的博客" # 网站名
theme = "PaperMod" # 如果前面选择了安装theme，可以填写
```
4.构建项目
终端执行hugo，构建静态网页
```
hugo
```
启用hugo服务器，查生成网页
```
hugo server
```
![alt text](/images/image-19.png)

### 远程配置

1. 访问github，创建托管项目

在浏览器地址栏输入, 注意需要魔法上网
```
github.com
```
没有github账户，可以问ai或搜索注册github的账户

有账户登录后，点击New创建一个新项目

![alt text](/images/image-3.png)

项目名称得是，“(github的账户名.github.io)”, 这里由于已经创建了该项目，同名所以出现警告
![alt text](/images/image-4.png)
其余填写地方默认就行，之后可以修改，然后创建项目
![alt text](/images/image-5.png)

2. 配置Github Actions
进入项目主页后，点击Setings
![alt text](/images/image-6.png)
进入以后，找到Pages点击
![alt text](/images/image-7.png)
找到Build and deployment，在source选择Github Actions
![alt text](/images/image-8.png)
改为
![alt text](/images/image-9.png)

3. 配置工作流
返回到项目主页，点击Actions
![alt text](/images/image-10.png)
点击New workflow
![alt text](/images/image-11.png)
在搜索栏输入static, enter（注意如果你有其他的项目要配置工作流也可以在这里查找现成的）
![alt text](/images/image-12.png)
选择static HTML，点击configure
![alt text](/images/image-13.png)
将以下内容复制到static.yml中，怎么配置，不在本教程的范围中。感兴趣可以自行了解ci/cd工作原理，非常简单高效

```yml
name: Deploy Hugo site to GitHub Pages
# 只要你往main分支push，就启动工作流
on:
  push:
    branches:
      - main
# 配置好权限以及身份id
permissions:
  contents: read
  pages: write
  id-token: write
# 正式工作流

jobs:
  build:
  #创建一台ubuntu机器
    runs-on: ubuntu-latest
  # 拉取你的代码到github pages云服务器上
    steps: 
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0
      # 安装hugo    
      - name: Setup hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true
      # 使用hugo构建网站
      - name: Build
        run: hugo --minify
      #上传构建结果
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  # 等待构建完毕
  deploy: 
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.pages_url}}
    runs-on: ubuntu-latest
    needs: build
    # 将构建的public发布到github pages CDN中
    steps:
      - name: Deploy to Github Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
提交更改
![alt text](/images/image-14.png)
填写commit message 记录自己的工作内容，方便之后的修改和查找，描述信息，最好简述修改内容就行
![alt text](/images/image-16.png)
查看主页文件出现.github成功，可以查阅生成static
![alt text](/images/image-17.png)
注意可以在配置好后Build and deployment，直接配置static。官方也提供教程通过配置GitHub Pages Jekyll
![alt text](/images/image-18.png)



### 验证工作流
1. 拉取远程仓库，并提交本地分支
回到本地创建的项目中，执行

```
git init # 初始化git
git add . # 添加所有本地文件
git commit -m "first deploy"
git branch -M main# 命令分支名称为main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git # 设置远程库
 # 远程分支中已经创建内容，与本地分支的内容不一致，拉取远程分支内容到本地
git pull --rebase origin main
git push -u origin main# 上传本地分支

```

2. 查看github上的自动部署情况
访问github项目地址

等待部署，显示黄色
![alt text](/images/image-20.png)
部署成功
![alt text](/images/image-21.png)
可以点击主页Actions，查看详细的部署情况
![alt text](/images/image-22.png)

点击Deploy Hugo site to GitHub Pages，查看每一次的提交部署情况，查看成功部署记录和失败出错的地方
![alt text](/images/image-23.png)
![alt text](/images/image-24.png)

## 总结

1. 不用自己买服务器，买域名，配置服务器，搭建网站麻烦，而且不用担心，安全和带宽的问题，还可以开通评论，有评论网站国内审批是比较麻烦的。
2. 配置的是静态网站，没有实现太复杂的交互逻辑，之后可以改变一下
3. 只用写markdown文件，具体的交互，数据管理，外观，都可以通过简单的配置实现，不用自己一步一步的实现。对新人友好。同时复杂的交互逻辑也可以写，正反都是html/js/css那套逻辑。
4. 作为技术blog够用了
