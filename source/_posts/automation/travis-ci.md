---
title: travis自动构建和部署vuePress文档到Github Pages
p: /automation/travis-ci
date: 2019-08-16 11:51:58
categories: 
    - ['自动化工程', '构建工具']
tags: 
    - [自动化工程]
    - [构建]
    - [travis]
---

Github中每个仓库都可以设置Github pages。仓库会有一个特殊的分支，gh-pages，该分支会在特殊位置发布，具体的访问地址为username.github.io/my-repository-name,访问的页面为gh-pages中的index.html。

例如我的仓库，[css-trick](https://github.com/vandvassily/css-trick)，Github Pages的访问地址为 [http://vandvassily.github.io/css-trick/](http://vandvassily.github.io/css-trick/),因为我配置了自己的域名 [vandvassily.cn](vandvassily.cn)，所以浏览器地址栏的域名显示为[http://vandvassily.cn/css-trick/](http://vandvassily.cn/css-trick/)

## 创建仓库

在Github中新建仓库，克隆仓库到本地开发环境

## 初始化vuePress项目

参考[vuePress官方文档](https://vuepress.vuejs.org/zh/guide/)，需要注意的是在`docs/.vuepress/config.js`中设置正确的`base`

``` javascript
#docs/.vuepress/config.js

module.exports = {
    base: '/css-trick/', // 仓库名称
    ...
}
```

在`package.json`文件中添加两条脚本

```json
    ...
    "scripts": {
        ...
        "docs:dev": "vuepress dev docs",
        "docs:build": "vuepress build docs"
    },
    ...
```

添加本地依赖`yarn add -D vuepress`

## 配置.travis.yml文件

在项目的根目录下，新建`.travis.yml`文件，

```yaml
language: node_js
node_js:
    - 10
cache: yarn
install:
    - yarn
script:
    - yarn docs:build
after_success:
    - cd docs/.vuepress/dist
    - git init
    - git config --global user.name "${U_NAME}"
    - git config --global user.email "${U_EMAIL}"
    - git add -A
    - git commit -m 'deploy'
    - git push -f "https://${access_token}@${GH_REPO}" master:${DEPLOY_BRANCH}

branches:
  only:
    - master
```

## 获取Github token

Github中选择`Settings-Developer settings-Personal access tokens`，创建新的token，除了删除选项，其余都可以勾选上

## 关联travis和github

在[travis-ci](https://travis-ci.org/)中关联Github账号

## travis中项目环境配置

在`travis-ci`中，账户选择`Settings`，选择想要自动化的repo，点击`Settings`，在`Environment Variables`中，新增5个环境变量`access_token`(github token)、`U_NAME`(git用户名)、`U_EMAIL`(git邮箱)、`GH_REPO`(项目仓库地址)、`DEPLOY_BRANCH`(发布分支，value为gh-pages)

## 本地推送代码，查看效果

本地push代码到github，travis在我们push完代码后，会自动执行`.travis.yml`中的指令

<fancybox>![avatar](travis2.png)</fancybox>
