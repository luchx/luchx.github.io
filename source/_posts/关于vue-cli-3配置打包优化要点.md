---
title: 关于vue-cli 3配置打包优化要点
date: 2019-04-21 17:41:32
categories: vue优化
tags: [vue, 优化]
---

## 关于vue-cli 3配置打包优化要点

首先说下我目前已经做的优化点,本文是在此基础上做的进一步优化:

- 配置路由懒加载,封装了异步组件引入的方法,接收一个地址做参数
```js
/**
 *  返回异步组件
 * @tips 请注意页面只能挂载在views文件下,非此路径请勿使用
 */
const AsyncComponentHook = (path: String): Function => (): any => {
    // 通过 webpack 的内联注释,设置模块名
    let component = import(/* webpackChunkName: "view-[request]" */ `@/views/${path}.vue`);
    component.catch((e) => {
        console.error(e);
    });
    return component;
};
```

- 配置代码压缩
```js
    // 安装
    npm install uglifyjs-webpack-plugin

    // 在vue.config.js文件中添加配置
    configureWebpack: config => {
        if (process.env.NODE_ENV === 'production') {
            // 为生产环境修改配置...
            config.plugins.push(
                //生产环境自动删除console
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
        }
    },

```

- 配置引用别名

- 设置插件的按需引入,本文使用的是`element-ui`，[点击查看详情](https://element.eleme.cn/#/zh-CN/component/quickstart)

...

> 经过一些基础的配置后,我们来看下目前打包后的效果。

从下图可以看到,打包出来后最大的包有`1.33M`。然后再看下请求,哇,`217`个请求、首页下载需要`3.2M`。

<div align="center">
    <img src="analyze-unoptimize.png" width="100%" alt="预览图" />
    <img src="unoptimize.png" width="100%" alt="预览图" />
</div>

...

好吧，开始折腾

### 1. 优化scss配置文件的引入

我们在搭建项目的过程中经常性的会将一些scss配置文件抽离出来，例如主题色等，然后在每个需要的组件中引入。这样会显得很繁琐，我们可以借助sass-loader帮我们进行预处理，
这样我们就不用在每一个页面都进行引入样式，就能直接引用。

> 例如我们的样式文件目录下有一个theme.scss，我们可以在`vue.config.js`中作如下处理
```js
// vue.config.js 配置
module.exports = {
  css: {
    // css预设器配置项
    loaderOptions: {
        // pass options to sass-loader
        sass: {
            // 引入全局变量样式,@使我们设置的别名,执行src目录
            data: `
                @import "@/stylePath/theme.scss";
            `
        }
    },
  },
}
```

> 通过以上配置,就可以在组件模板中注释以下代码
```css
<style lang="scss">
   /* @import "@/stylePath/theme.scss"; */
</style>
```

### 2. 针对请求数进行优化

我们发现请求数增多是因为我们页面预先渲染了其它组件，会在html页面中插入像这样的东西`<link rel="prefetch">`，这该怎么优化呢？

> 首先我们先看下`vue.config.js`的官方文档，[点击前往](https://cli.vuejs.org/zh/guide/html-and-static-assets.html#prefetch)。
官方说明： <link rel="prefetch"> 是一种 resource hint，用来告诉浏览器在页面加载完成后，利用空闲时间提前获取用户未来可能会访问的内容。

所以我们添加如下配置
```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // 移除 prefetch 插件
    config.plugins.delete('prefetch')
    // 移除 preload 插件
    config.plugins.delete('preload');
  }
}
```

### 3. 公用代码提取，使用cdn加载

对于`vue`, `vuex`, `vue-router`，`axios`等我们可以利用`wenpack`的`externals`参数来配置，这里我们设定只需要在生产环境中才需要使用:
```js
// vue.config.js
const isProduction = process.env.NODE_ENV === 'production';
const cdn = {
    css: [],
    js: [
        'https://cdn.bootcss.com/vue/2.5.17/vue.runtime.min.js',
        'https://cdn.bootcss.com/vue-router/3.0.1/vue-router.min.js',
        'https://cdn.bootcss.com/vuex/3.0.1/vuex.min.js',
        'https://cdn.bootcss.com/axios/0.18.0/axios.min.js',
    ]
}
module.exports = {
    chainWebpack: config => {
        // 生产环境配置
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
            // 用cdn方式引入
            config.externals = {
                'vue': 'Vue',
                'vuex': 'Vuex',
                'vue-router': 'VueRouter',
                'axios': 'axios'
            }
        }
    },
}
```

接着修改`html`文件，添加注入代码
```html
<!DOCTYPE html>
<html lang="zh">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
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

然后打包重启，我们再来看下目前的变化。

嗯，真香~从下图可以看到,打包出来后最大的包文件只有`775kb`。然后再看下请求,哇,`43`个请求、首页下载只需要`1.4M`。

可以看出，我们这一系列的操作后请求数减少了`174`个，首页渲染减少了`1.8m`,真是可喜可贺

<div align="center">
    <img src="analyze-optimize.png" width="100%" alt="预览图" />
    <img src="optimize.png" width="100%" alt="预览图" />
</div>

最后，附上完整的`vue-config.js`文件
```js
const path = require("path");
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

const isProduction = process.env.NODE_ENV === 'production';
const cdn = {
    css: [],
    js: [
        'https://cdn.bootcss.com/vue/2.5.17/vue.runtime.min.js',
        'https://cdn.bootcss.com/vue-router/3.0.1/vue-router.min.js',
        'https://cdn.bootcss.com/vuex/3.0.1/vuex.min.js',
        'https://cdn.bootcss.com/axios/0.18.0/axios.min.js',
    ]
}

function resolve(dir) {
    return path.join(__dirname, dir)
}

module.exports = {
    // 基本路径
    baseUrl: './',
    // 输出文件目录
    outputDir: 'dist',
    // 放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录。
    // assetsDir: "./",
    // 指定生成的 index.html 的输出路径 (相对于 outputDir)。也可以是一个绝对路径
    indexPath: './',
    // eslint-loader 是否在保存的时候检查
    lintOnSave: true,
    // webpack配置
    // see https://github.com/vuejs/vue-cli/blob/dev/docs/webpack.md
    chainWebpack: config => {
        config
            .entry('index')
            .add('babel-polyfill')
            .end();
        // 配置别名
        config.resolve.alias
            .set("@", resolve("src"))
            .set("@img", resolve("src/assets/images"))
            .set("@css", resolve("src/assets/styles/css"))
            .set("@scss", resolve("src/assets/styles/scss"));
        // 生产环境配置
        if (isProduction) {
            // 删除预加载
            config.plugins.delete('preload');
            config.plugins.delete('prefetch');
            // 压缩代码
            config.optimization.minimize(true);
            // 分割代码
            config.optimization.splitChunks({
                chunks: 'all'
            })
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
            // 用cdn方式引入
            config.externals = {
                'vue': 'Vue',
                'vuex': 'Vuex',
                'vue-router': 'VueRouter',
                'axios': 'axios'
            }
            // 为生产环境修改配置...
            config.plugins.push(
                //生产环境自动删除console
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
    // 生产环境是否生成 sourceMap 文件
    productionSourceMap: false,
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
                // 引入全局变量样式
                data: `
                    @import "@/stylePath/theme.scss;
                `
            }
        },
        // 启用 CSS modules for all css / pre-processor files.
        modules: false,
    },
    // use thread-loader for babel & TS in production build
    // enabled by default if the machine has more than 1 cores
    parallel: require('os').cpus().length > 1,
    devServer: {
        port: 8888,  // 端口
        open: true, // 自动开启浏览器
        compress: false, // 开启压缩
        overlay: {
            warnings: true,
            errors: true
        }
    },
}
```

以上就是我针对打包后做的优化处理，当然还有其它优化点，比如开启`gzip`压缩，不过这个需要后台服务器支持，所以暂不配置。如果你还有其它优化点，欢迎一起讨论！


