---
title: Travis自动部署Hexo博客
date: 2019-06-16 21:57:39
tags: 博客
---

#### 1.<a href="https://github.com/">Github</a>中创建存储Hexo博客项目源码的仓库（空仓）

#### 2.使用Github账户登录<a href="https://www.travis-ci.org/">Travis</a>

#### 3.选择并激活存储Hexo博客项目源码的仓库

#### 4.在Github账户settings->developer setting->Personal access tokes 生成token

![1560694229552](Travis自动部署Hexo博客/1560694229552.png)

#### 5.配置Travis的环境变量

![1560694061229](Travis自动部署Hexo博客/1560694061229.png)

#### 6.在Hexo博客根目录下添加<a href="https://github.com/fangchieh/blog/blob/master/.travis.yml">.travis.yml</a>配置文件

```yml
language: node_js # 设置语言

node_js: stable # 设置相应版本

cache:
    directories:
        - node_modules # 设置缓存，会在构建的时候快一些

# before_install:
    # - npm install hexo-cli -g

install:
    - npm install # 安装hexo及插件
    #- npm install -g gulp 

script:
    - hexo clean # 清除
    - hexo g # 生成
  

after_script:

    - cd public 
    - git init
    - git config user.name "fangchieh"
    - git config user.email "358545276@qq.com"
    - git add .
    - git commit -am "Travis CI Auto Builder :$(date '+%Y-%m-%d %H:%M:%S' -d '+8 hour')"
    - git push -f ${GH_REF} master  
    - git push -f ${CODING} master
    
    
branches:
    only:
        - master #只监测master分支,这是我自己的博客，所以就用的master分支了。
env:
    global:
        - GH_REF: https://fangchieh:${travis}@github.com/fangchieh/fangchieh.github.io.git #设置GH_REF，注意更改yourname,travis:就是我们在travis-ci仓库中配置的环境变量
        - CODING: https://fangchieh:${coding_travis}@git.dev.tencent.com/fangchieh/fangchieh.coding.me.git
```

#### 7.在本地提交.travis.yml更新后，将整个博客项目push到新建的GitHub空仓

<font color="red">注意</font>：必须先将博客项目初始化git项目

```bash
cd myblog (博客根目录)
git init . (初始化成git项目)
git add .  (将所有博客文件添加到暂存区并跟踪)
git commit -m "描述" (提交暂存区的文件到版本库)
git remote add origin git@github.com/youname/空仓名称.git (为本地git项目添加远程仓库地址)
git push -u origin master  （将本地项目推送到远程仓库的master主分支）
```

#### 8.当travis检测到源码仓库有push时，就会自动构建（build）

![1560696215594](Travis自动部署Hexo博客/1560696215594.png)

![1560696151584](Travis自动部署Hexo博客/1560696151584.png)

