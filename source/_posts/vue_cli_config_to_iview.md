---
title: Vue-cli 3.0 配置iview
---

###  (一) iview 配置自定义主题

按照[iview官网](https://www.iviewui.com/)配置会出现如下错误

```js

    Module build failed (from ./node_modules/less-loader/dist/cjs.js):
    // https://github.com/ant-design/ant-motion/issues/44
    .bezierEasingMixin();
    ^
    Inline JavaScript is not enabled. Is it set in your options?
    in /el-admin/node_modules/iview/src/styles/color/bezierEasing.less (line 111, column 0)

```

上述错误需要将less的javascriptEnabled设置为true，接下来我们在vue-cli3中中进行配置

配置如下：

1. iview创建自定义主题样式文件，可以参考官网的案例 [查看](https://www.iviewui.com/docs/guide/theme)

2. 配置vue.config.js文件

```js

    module.exports = {
        css: { // 配置css模块
            loaderOptions: { // 向预处理器 Loader 传递配置选项
                less: { // 配置less（其他样式解析用法一致）
                        javascriptEnabled: true // 设置为true
                }
            }
        }
    }

```

以上的配置你也可以参考VueCli文档 [查看](https://www.iviewui.com/docs/guide/theme)

通过以上的配置，就可以解决iview的自定义主题的错误问题。

### (二) iview 配置按需引入组件

按照iview官网提供的配置进行按需引入会在IE下报错

解决方法: (适用于vue-cli3 构建的项目)

在项目根目录创建vue.config.js 写入如下配置：

```js

    module.exports = { 
        chainWebpack: config => { 
            // ie报错无效字符 添加该配置项 解决该问题 
            config.module
                .rule('iview') 
                .test(/iview.src.*?js$/) 
                .use('babel') 
                .loader('babel-loader') 
                .end() 
            } 
    }

```