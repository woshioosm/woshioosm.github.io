---
title: 基于github 搭建hexo+next博客
categories: hexo
tags:
 - hexo
 - next
---

## 概要
1. 搭建本地的调试环境
1. 基于github 发布hexo+next风格的博客
2. 基于github action， 进行自动的编译和发布

<!--more-->

## 基础环境安装
- 安装git
- 安装nodejs
- 安装npm

## 安装hexo
``` bash
$ npm install -g hexo-cli
$ hexo init blog
$ cd blog
$ npm install
```

## 安装next主题
下载netx
``` bash
$ cd theme
$ git clone https://github.com/theme-next/hexo-theme-next
```
修改blog目录下的_config.yml
``` yml
theme: next
```
本地部署
``` bash
$ hexo server
```
本地浏览
```
localhost:4000
```

## 添加功能
### 添加“标签” ,"分类"
1. 用命令生成文件
``` bash
$ hexo new page "tags"
$ hexo new page "categories"
```
2. 打开生成的index.md文件，title和date是默认生成的，增加type即可
``` properties
---
title: tags
date: 2019-06-25 19:16:17
type: "tags"
---
 
---
title: categories
date: 2019-06-25 19:16:17
type: "categories"
---
```

### 添加“搜索”

1. 在blog目录下安装插件 (接下来的自动发布 还需要加入到workflow中)
``` bash
npm install hexo-generator-search --save
```
2. 在blog的_config.yml 最后添加
``` yml
search:
  path: search.xml
  field: post
  format: html
  limit: 100
```

## 上传github
新建自己的github.io 项目， 将工程推送到github上
>next目录应该是一个引用，建议自己拷贝一份引入

## CNAME
如果自己的项目有域名解析关联，建立CNAME文件，文件中写入你的域名，放置在source目录中

## 自动发布
- 通过github action功能，每次上传新文件或者修改样式，都会重新编译生成新的页面
- 建议建立一个新的repo或者分支作为发布的目标
- 这里将master 设置为发布的分支， dev为维护blog源码和博客md的分支

操作流程如下：
1. 建立一对公私钥，使得自动部署有权限发布
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/GitHub-actions-deploy
```
2. 在 github-Settings-Deploy keys 中填入刚才生成的pub文件内容
3. 在github-Settings-Secrets 中填入刚才生成的私钥文件，secret的name命名为HEXO_DEPLOY_PRI（随意，但需要和后文的workflow匹配）
4. blog目录的_config.yml中添加(指定好要部署的位置)
``` yml
deploy:
  type: git
  repo: git@xxxxxx
  branch: master
```
5. github中打开工程的Actions-new workfolw （建立在dev分支中）
``` yml
name: CI
on:
  push:
    branches:
      - dev
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v1
        with:
          ref: dev
      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          version: ${{ matrix.node_version }}
      - name: Setup hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.HEXO_DEPLOY_PRI }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "lujiahao0708@gmail.com"
          git config --global user.name "lujiahao0708"
          npm install hexo-cli -g
          npm install --save hexo-deployer-git
          npm install hexo-generator-searchdb --save
          npm install
      - name: Hexo deploy
        run: |
          hexo clean
          hexo d
```
6. 保存这一切，每当发生push时，都会触发这个workflow 编译到我的master分支中
7. 在settings首页的GitHub Pages中指定到master即可访问
8. 配置可以参考
``` url
https://github.com/woshioosm/woshioosm.github.io  （dev分支）
```