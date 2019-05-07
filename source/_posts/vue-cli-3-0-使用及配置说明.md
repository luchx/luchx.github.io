---
title: vue-cli@3.0 使用及配置说明
date: 2019-05-07 21:44:38
categories: 个人随笔
tags: [vue, vue-cli@3.0, 教程]
---

> 使用vue-cli3已经有相当一段时间了，一直没怎么去注意其中的配置，所以趁着这段时间总结下应用过程中的一些经验，本文是从安装到开发使用的一个过程讲解，也可以说是新手向的文章，文字有点多，请耐心观看。

### （一）安装：
1、下载安装node: 登陆[node官网](https://nodejs.org/en/download/) 并选择自己合适的node版本进行安装；

2、安装vue-cli脚手架工具
```bash
npm install -g @vue/cli
# OR
# 推荐使用
npm install -g yarn # 如果已有安装，此步骤不需要
yarn global add @vue/cli 
```

### （二）创建一个项目：
```bash
vue create my-project
# OR
vue ui
```

- 选择合适的配置
```bash
# 版本信息
Vue CLI v3.7.0 
? Please pick a preset:
# 基础配置
  vue-cli3-demo (vue-router, vuex, sass, babel, typescript, unit-mocha)
  default (babel, eslint)
# 自定义配置，这里我们选择自定义选项，点击回车
> Manually select features
```

- 选择需要的插件及编译工具
```bash
? Check the features needed for your project:
# 代码转换，可以让你更好的书写前沿技术
 (*) Babel
# JavaScript 的一个超集，好东西用起来
 (*) TypeScript
# PWA支持,不要求使用可以不选
 ( ) Progressive Web App (PWA) Support
 (*) Router
 (*) Vuex
# css预编译器
 (*) CSS Pre-processors
# 代码格式化
 (*) Linter / Formatter
# 书写单元测试用的，不要求使用可以不选
>(*) Unit Testing
 ( ) E2E Testing
```

- 接下来的配置选项
```bash
# 是否使用class风格进行组件开发
? Use class-style component syntax? Yes
# 使用 babel 对 TypeScript 代码进行编译转换
? Use Babel alongside TypeScript for auto-detected polyfills? Yes

? Use history mode for router? (Requires proper server setup for index fallback in production) Yes
# 选择css预编译，这里我们选择Sass/SCSS
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with node-sass)
# 选择代码格式化配置
? Pick a linter / formatter config: Standard
# 代码检查方式
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)Lint on save
# 选择单元测试工具
? Pick a unit testing solution: Mocha
# 是否将配置放在单独的文件中
? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? In dedicated config files
# 是否将此次配置保存
? Save this as a preset for future projects? No
```

- 最后
```bash
cd my-project
npm run install
npm run serve
# OR 推荐使用
yarn install
yarn serve
```

### （三）目录结构

<div align="center">
    <img src="https://user-gold-cdn.xitu.io/2019/5/8/16a932222c8aa72d?w=318&h=853&f=png&s=26835" alt="预览图" />
</div>

### （四）环境变量配置

环境变量说明
```
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略
```

1. 新建环境变量 .env.development.test 用于测试环境
并添加如下代码
```
NODE_ENV='development'
VUE_APP_URL='你的测试环境域名'
```

> 只有以 VUE_APP_ 开头的变量会被 webpack.DefinePlugin 静态嵌入到客户端侧的包中。你可以在应用的代码中这样访问它们：
```js
console.log(process.env.VUE_APP_URL)
```

2. 修改 `package.json`,并在`scripts`里面添加如下代码
```js
"serve:test": "vue-cli-service serve --mode development.test",
```

3. 如果项目中有使用到公共环境变量，为了避免在每个.env文件中配置，我们也可以在`vue.config.js`里面进行配置

在设置之前我们先来看下2.0时代的环境变量配置，之前在 prod.env.js 中我们会如下配置：
```js
'use strict'
module.exports = {
    NODE_ENV: '"production"'
}
```

显然这样的配置我们目前不能修改，因为目前的配置文件只有`vue.config.js`，所以我们添加如下代码，进行公共环境变量的设置。

```js
// vue.config.js
module.exports = {
    chainWebpack: config => {
        // 添加环境变量 
        config.plugin("define")
            .tap(args => {
                args[0]["process.env"].VUE_APP_ENVBANE = JSON.stringify("环境变量值")
                return args;
            });
    },
}

```

### （五）添加配置文件 vue.config.js

[vue.config.js](https://cli.vuejs.org/zh/) 是一个可选的配置文件，如果项目的 (和 package.json 同级的) 根目录中存在这个文件，那么它会被 @vue/cli-service 自动加载。你也可以使用 package.json 中的 vue 字段，但是注意这种写法需要你严格遵照 JSON 的格式来写。
```json
// vue.config.js
module.exports = {
    // baseUrl从 Vue CLI 3.3 起已弃用，请使用publicPath。
    // baseUrl:'./', 
    // 配置sub-path后访问路径为https://xxx-path/sub-path/#/
    publicPath: process.env.NODE_ENV === 'production' ? '/sub-path/' : '/',
    // 输出文件路径，默认为dist
    outputDir: 'dist',  
    // 放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录。
    assetsDir: '', 
    // 指定生成的 index.html 的输出路径 (相对于 outputDir)。也可以是一个绝对路径
    indexPath: '',
    // 配置多页应用
    pages: {
        index: {
            // page 的入口
            entry: 'src/index/main.js',
            // 模板来源
            template: 'public/index.html',
            // 在 dist/index.html 的输出
            filename: 'index.html',
            // 当使用 title 选项时，
            // template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
            title: 'Index Page',
            // 在这个页面中包含的块，默认情况下会包含
            // 提取出来的通用 chunk 和 vendor chunk。
            chunks: ['chunk-vendors', 'chunk-common', 'index']
        },
        // 当使用只有入口的字符串格式时，
        // 模板会被推导为 `public/subpage.html`
        // 并且如果找不到的话，就回退到 `public/index.html`。
        // 输出文件名会被推导为 `subpage.html`。
        subpage: 'src/subpage/main.js',
    },
    lintOnSave: true,  // 保存时 lint 代码
    // css相关配置
    css: {
        // 是否使用css分离插件 ExtractTextPlugin
        extract: true,
        // 开启 CSS source maps?
        sourceMap: false,
        // css预设器配置项
        loaderOptions: {
            // pass options to sass-loader
            sass: {
                // 自动注入全局变量样式
                data: `
                    @import "src/你的全局scss文件路径";
                `
            }
        },
        // 启用 CSS modules for all css / pre-processor files.
        modules: false,
    },
    // 生产环境是否生成 sourceMap 文件
    productionSourceMap: false,
    //是否为 Babel 或 TypeScript 使用 thread-loader。该选项在系统的 CPU 有多于一个内核时自动启用，仅作用于生产构建。
    parallel: require('os').cpus().length > 1,
    // 所有 webpack-dev-server 的选项都支持
    devServer: {
        port: 8080, // 配置端口
        open: true, // 自动开启浏览器
        compress: true, // 开启压缩
        // 设置让浏览器 overlay 同时显示警告和错误
        overlay: {
            warnings: true,
            errors: true
        },
        // 设置请求代理
        proxy: {
            '/api': {
                target: '<url>',
                ws: true,
                changeOrigin: true
            },
            '/foo': {
                target: '<other_url>'
            }
        }
    },
}
```

### （六）修改`webpack`配置信息

`vue-cli3.0`的版本已经将`webpack`的配置整合进`vue.config.js`里面了

```js
// 安装 babel-polyfill
// npm install babel-polyfill 或者 yarn add babel-polyfill
// 安装 uglifyjs-webpack-plugin
// npm install uglifyjs-webpack-plugin -D 或者 yarn add uglifyjs-webpack-plugin -D

// vue.config.js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
    chainWebpack: config => {
        // 引入babel-polyfill
        config
            .entry('index')
            .add('babel-polyfill')
            .end();
        // 添加文件路径别名
        config.resolve.alias.set("@", resolve("src"));
        if (isProduction) {
            // 生产环境注入cdn
            config.plugin('html')
                .tap(args => {
                    args[0].cdn = cdn;
                    return args;
                });
        }
    },
    configureWebpack: config => {
        if (isProduction) {
            // 为生产环境修改配置...
            config.plugins.push(
                //添加代码压缩工具，及设置生产环境自动删除console
                new UglifyJsPlugin({
                    uglifyOptions: {
                        compress: {
                            warnings: false,
                            drop_debugger: true,
                            drop_console: true,
                        },
                    },
                    sourceMap: false,
                    parallel: true,
                })
            );
        } else {
            // 为开发环境修改配置...
        }
    },
}
```

3. 分离第三方插件，引入cdn配置

这里介绍一个开源的cdn库 https://www.bootcdn.cn/

```js
// vue.config.js
const isProduction = process.env.NODE_ENV === 'production';
const cdn = {
    css: [],
    js: [
        'https://xxx-cdn-path/vue.runtime.min.js',
        'https://xxx-cdn-path/vue-router.min.js',
        'https://xxx-cdn-path/vuex.min.js',
        'https://xxx-cdn-path/axios.min.js',
    ]
}

module.exports = {
    configureWebpack: config => {
        if (isProduction) {
            // 用cdn方式引入,分离第三方插件
            config.externals = {
                'vue': 'Vue',
                'vuex': 'Vuex',
                'vue-router': 'VueRouter',
                'axios': 'axios'
            }
        } else {
            // 为开发环境修改配置...
        }
    },
}
```
修改html文件
```html
<!DOCTYPE html>
<html lang="zh">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" href="<%= BASE_URL %>static/favicon.ico" type="image/x-icon" />
  <link rel="shortcut icon" href="<%= BASE_URL %>static/favicon.ico" type="image/x-icon" />
  <title>my-project</title>
  <!-- 使用CDN的CSS文件 -->
  <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css) { %>
    <link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="preload" as="style">
    <link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="stylesheet">
  <% } %>
  <!-- 使用CDN的JS文件 -->
  <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
    <link href="<%= htmlWebpackPlugin.options.cdn.js[i] %>" rel="preload" as="script">
  <% } %>
</head>

<body>
  <noscript>
    <strong>We're sorry but eye-admin doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
  </noscript>
  <div id="app"></div>
  <!-- built files will be auto injected -->
  <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
    <script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
  <% } %>
</body>

</html>
```

### （七）关于打包后请求数的优化点Preload and Prefetch
首先我们看一张图

<div align="center">
    <img src="https://user-gold-cdn.xitu.io/2019/5/8/16a9323696acad14?w=1271&h=765&f=png&s=49982" alt="预览图" />
</div>

从图中我们可以看出首次加载的资源非常多，有217个请求，为什么会这样呢？

查看[官方文档](https://cli.vuejs.org/zh/guide/html-and-static-assets.html#插值)，可以得知： 
```
<link rel="preload"> 是一种 resource hint，用来指定页面加载后很快会被用到的资源，所以在页面加载的过程中，我们希望在浏览器开始主体渲染之前尽早 preload。

默认情况下，一个 Vue CLI 应用会为所有初始化渲染需要的文件自动生成 preload 提示。

这些提示会被 @vue/preload-webpack-plugin 注入，并且可以通过 chainWebpack 的 config.plugin('preload') 进行修改和删除。
```

```
<link rel="prefetch"> 是一种 resource hint，用来告诉浏览器在页面加载完成后，利用空闲时间提前获取用户未来可能会访问的内容。

默认情况下，一个 Vue CLI 应用会为所有作为 async chunk 生成的 JavaScript 文件 (通过动态 import() 按需 code splitting 的产物) 自动生成 prefetch 提示。

这些提示会被 @vue/preload-webpack-plugin 注入，并且可以通过 chainWebpack 的 config.plugin('prefetch') 进行修改和删除。
```

所以修改`vue.config.js`文件
```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // 移除 prefetch 插件
    config.plugins.delete('preload');
    config.plugins.delete('prefetch');
  }
}
```


### （八）总结
vue-cli3在项目配置上精简了很多，而且它也提供了很多配置选项，满足定制化需要。各种配置也特别贴心，可以按照自己项目的需要进行自定义修改，大大减少了提升了开发的工作效率。

以上就是本文的全部内容，我们已经将vue-cli3.0从安装到修改配置的过程讲解结束了，希望对大家的学习有所帮助
