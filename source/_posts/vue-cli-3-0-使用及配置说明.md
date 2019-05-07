---
title: vue-cli@3.0 使用及配置说明
date: 2019-05-07 21:44:38
categories: 个人随笔
tags: [vue, vue-cli@3.0, 教程]
---

### 安装：
1、下载安装node: 登陆node官网 https://nodejs.org/en/download/ 并选择自己合适的node版本；

2、安装vue-cli脚手架工具
```bash
npm install -g @vue/cli
# OR
npm install -g yarn # 如果已有安装，此步骤不需要
yarn global add @vue/cli
```

### 创建一个项目：
```bash
vue create my-project
# OR
vue ui
```

选择合适的配置
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

选择需要的插件及编译工具
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

接下来的配置选项
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

最后
```bash
cd my-project
npm run serve
OR
yarn serve
```

### 目录结构

<div align="center">
    <img src="assets.png" width="100%" alt="预览图" />
</div>