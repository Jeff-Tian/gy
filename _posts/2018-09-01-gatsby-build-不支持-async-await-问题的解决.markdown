---
title: gatsby build 不支持 async await 问题的解决
layout: post
date: '2018-09-01 16:48:06 +0800'
categories: 程序员日常
---

### 症状：
使用 `gatsby` 第一版创建的项目，在 `gatsby develop` 时运行正常，但是在发布时 `gatsby build` 时报错：
```
 Error: component---src-pages-index-js-c938d5b5226f72f5900f.js from UglifyJs
  SyntaxError: Unexpected token operator «*», expected punc «(» [./src/pages/index.js:20,19]
```

### 分析：
找到源代码这一行，发现是一个 `async` 函数，看来第一版的 `gatsby` 的 `webpack` 默认配置并不支持 `ES 6`。

### 找解决办法：
看了几篇`github` 聊天记录，有很多人碰到这个问题，最终在[这篇聊天记录](https://github.com/gatsbyjs/gatsby/issues/3931#issuecomment-364414141)里找到了可以说是最优雅的解决办法了：

- 首先安装一个`babel`插件：
```bash
yarn add babel-plugin-transform-regenerator
```

- 然后在`gatsby-node.js`文件中加入以下内容：
```javascript
 exports.modifyBabelrc = ({ babelrc }) => ({
    ...babelrc,
    plugins: babelrc.plugins.concat(['transform-regenerator']),
  })
```

- 然后`gatsby build`就可以成功啦！