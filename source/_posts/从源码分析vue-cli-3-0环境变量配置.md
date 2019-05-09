---
title: 从源码分析vue-cli@3.0环境变量配置
date: 2019-05-09 19:30:01
categories: 源码
tags: [环境配置, 源码分析]
---

> 查看[vue-cli文档](https://cli.vuejs.org/zh/guide/mode-and-env.html#示例：staging-模式)中有这么一句话：

```
只有以 VUE_APP_ 开头的变量会被 webpack.DefinePlugin 静态嵌入到客户端侧的包中。
```
