---
title: "自动发布白鹭引擎小游戏到微信平台 CI/CD"
date: 2021-07-14T18:30:36+08:00
categories:
 - 技术
tags:
 - Egret
 - WeChat
 - CI/CD
---

> 生命在于折腾

最近一段时间在使用白鹭 Egret Engine, 为了统一和方便，选择使用 GitHub Action 做 CI/CD，但在折腾的过程中，发现这并不简单。

首先要明确的事，微信小程序是有一个 `miniprogram-ci` 的包用于上传项目到平台，而且在微信后台看是不同的开发者，也就意味着可以使用这个bot打包不同的版本。(ci 工具支持最多 30 个bot) [文档*](https://developers.weixin.qq.com/miniprogram/dev/devtools/ci.html)

> npm install miniprogram-ci

有了这个 ci 工具其实解决了最大的难题，（尽管我们可以通过 gui 版的微信开发者工具上传，但很难处理 token 之类的问题）

下一个要解决的难题是，编译白鹭项目成微信，我想着这部分会很简单，安装一下白鹭引擎，再`egret publish --target wxgame`就解决了，但没想到这一步卡了很久。

---

得益于白鹭引擎是个在 GitHub 开源的项目，可以直接使用 [`actions/checkout`](https://github.com/actions/checkout) 拿到白鹭的项目，以及要把自己的项目 `checkout` 出来，

```
- name: Checkout Egret Engine
  uses: actions/checkout@v2
  with:
    repository: egret-labs/egret-core
    ref: "v5.4.1"
    path: "engine"
```

> 如果是私人项目 (private repo)，那需要在设置里生成一个 PAT (Personal Access Token) 并通过项目设置中的密钥 (Secrets) 传入到 ci 脚本中。

本以为事情到这里就结束了，使用白鹭的脚本打包，使用微信的 CI 上传，万事大吉。

然而，白鹭编译的过程中需要白鹭自己的 Compiler 去打包，一番研究，白鹭有把 egret compiler 发布到 npm 上，可以直接 `npm install @egret/eui-compiler@1.4.9` 安装。

但，这还不够，在运行的时候，会报缺少 `egret-webpack-bundler` 的错误，找了一圈发现也可以通过 `npm install @egret/egret-webpack-bundler` 安装。

本以为事情到这里就结束了 again。

发现还是会有报错，目录问题。 好在是个开源项目，直接读 `egret compiler` 的源码，竟然是写死的目录！于是只能通过各种 `mkdir`, `mv` 模拟成 egret compiler 希望使用的。

后面的事情就简单了，配置下微信 CI 工具，包括版本号，appID，密钥等信息

---

 > CI 脚本写的比较乱 :)

```yml
name: WxGame CI

on:
  push:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x]

    steps:

    - name: Step Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}

    - name: Checkout Egret Engine
      uses: actions/checkout@v2
      with:
        repository: egret-labs/egret-core
        ref: "v5.4.1"
        path: "engine"

    - name: Checkout Project
      uses: actions/checkout@v2
      with:
        repository: [repo-author/project]
        ref: "master"
        path: "./project/"
        token: ${{ secrets.PAT }}

    - name: Preparing miniprogram ci
      run: npm install miniprogram-ci @egret/egret-library-installer -g

    - name: "Rearrange folder"
      run: |
        mkdir -p /home/runner/.egret/engine/5.4.1
        mkdir -p /tmp/project
        mv ./engine/* /home/runner/.egret/engine/5.4.1/
        mv ./project/* /tmp/project/

    - name: Install egret compiler
      run: |
        mkdir -p /tmp/egret/compiler/EgretLauncher/download/EgretCompiler
        cd /tmp/egret/compiler/EgretLauncher/download/EgretCompiler
        npm install --prefix . @egret/egret-webpack-bundler@2.0.3 @egret/eui-compiler@1.4.9
        mv ./node_modules/@egret .
        cd @egret/egret-webpack-bundler
        npm install @egret/texture-merger-core

    - name: Building project
      run: |
        npm install --prefix /tmp/project/scripts uglify-js
        /home/runner/.egret/engine/5.4.1/tools/bin/egret publish /tmp/project --target wxgame
      env:
        EGRET_PATH: /home/runner/.egret/engine/5.4.1/
        XDG_CONFIG_HOME: /tmp/egret/compiler

    - name: Generate private key for upload
      run: echo "$WX_UPLOAD_PRIVATE_KEY" > private.key
      env:
        WX_UPLOAD_PRIVATE_KEY: ${{ secrets.WX_UPLOAD_PRIVATE_KEY }}

    - name: uploading project
      run: miniprogram-ci upload --pp /tmp/project/wxgame --pkp ./private.key --appid APP_ID --uv APP_VERSION --ud "${{ github.event.head_commit.message }}" -r 1 --enable-es6 true  --pt miniGame
```

