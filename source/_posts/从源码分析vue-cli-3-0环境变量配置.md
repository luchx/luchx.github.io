---
title: 从源码分析vue-cli@3.0环境变量配置
date: 2019-05-09 19:30:01
categories: 源码
tags: [环境配置, 源码分析]
---

### 前言

> 在开始之前，我们先来看下官方文档说明；查看[vue-cli文档](https://cli.vuejs.org/zh/guide/mode-and-env.html#示例：staging-模式)中有这么一句话：

```
只有以 VUE_APP_ 开头的变量会被 webpack.DefinePlugin 静态嵌入到客户端侧的包中。
```

由此我们可以知道，`vue-cli` 是有针对 `VUE_APP_` 这个变量进行配置。接下来让我们查看源码，看下`vue-cli`是怎么实现的。

### 源码分析

查看根路径的 `package.json` 文件我们可以知道，当执行 `npm run serve` 时会执行 `vue-cli-service`；所以我们找到该文件的执行地址`node_modules/@vue/cli-service`。

分析其 `package.json` 我们知道，当我们执行 `vue-cli-service` 时候会调用文件 `lib/Service.js` 中 `init` 函数：

```js
init (mode = process.env.VUE_CLI_MODE) {
    ...
    this.mode = mode

    // load mode .env
    if (mode) {
      this.loadEnv(mode)
    }
    // load base .env
    this.loadEnv()
    ...
  }
```

紧接着我们再来看`loadEnv`方法

```js
loadEnv (mode) {
    const logger = debug('vue:env')
    const basePath = path.resolve(this.context, `.env${mode ? `.${mode}` : ``}`)
    // 注意这里，不管你有没有定义它都会在后面加上一个.local的后缀，感觉这里不是很好
    const localPath = `${basePath}.local`

    const load = path => {
        try {
            const env = dotenv.config({ path, debug: process.env.DEBUG })
            dotenvExpand(env)
            logger(path, env)
        } catch (err) {
            // only ignore error if file is not found
            if (err.toString().indexOf('ENOENT') < 0) {
                error(err)
            }
        }
    }

    load(localPath)
    load(basePath)

    ...
}
```

这里会先调用 `dotenv`（位于 `node_modules/dotenv` ）的 `config` 函数，最终会返回这样的格式 `{ parsed: { YOUR_ENV_KEY: '你设定的环境变量值' } }`。
并且我们可以看到，这里已经将值存放到`process.env`中了。

```js
function config (options /*: ?DotenvConfigOptions */) /*: DotenvConfigOutput */ {
  ...

  try {
    // specifying an encoding returns a string instead of a buffer
    const parsed = parse(fs.readFileSync(dotenvPath, { encoding }), { debug })

    Object.keys(parsed).forEach(function (key) {
      if (!process.env.hasOwnProperty(key)) {
        process.env[key] = parsed[key]
      } else if (debug) {
        log(`"${key}" is already defined in \`process.env\` and will not be overwritten`)
      }
    })

    return { parsed }
  } catch (e) {
    return { error: e }
  }
}
```

这里再说下 `parse` 方法，会读取你设置的所有 `.env` 文件，然后将里面的数据转换成对象，简单来说就是 `以换行符号来循环，用正则匹配出内容`，最终形成以`{key: value}`的格式输出。

```js
const NEWLINE = '\n'
const RE_INI_KEY_VAL = /^\s*([\w.-]+)\s*=\s*(.*)?\s*$/
const RE_NEWLINES = /\\n/g

function parse (src /*: string | Buffer */, options /*: ?DotenvParseOptions */) /*: DotenvParseOutput */ {
  const debug = Boolean(options && options.debug)
  const obj = {}

  // convert Buffers before splitting into lines and processing
  src.toString().split(NEWLINE).forEach(function (line, idx) {
    // matching "KEY' and 'VAL' in 'KEY=VAL'
    const keyValueArr = line.match(RE_INI_KEY_VAL)
    // matched?
    if (keyValueArr != null) {
      const key = keyValueArr[1]
      // default undefined or missing values to empty string
      let val = (keyValueArr[2] || '')
      const end = val.length - 1
      const isDoubleQuoted = val[0] === '"' && val[end] === '"'
      const isSingleQuoted = val[0] === "'" && val[end] === "'"

      // if single or double quoted, remove quotes
      if (isSingleQuoted || isDoubleQuoted) {
        val = val.substring(1, end)

        // if double quoted, expand newlines
        if (isDoubleQuoted) {
          val = val.replace(RE_NEWLINES, NEWLINE)
        }
      } else {
        // remove surrounding whitespace
        val = val.trim()
      }

      obj[key] = val
    } else if (debug) {
      log(`did not match key and value when parsing line ${idx + 1}: ${line}`)
    }
  })

  return obj
}
```

然后我们再来看下 `dotenvExpand` 方法，找到它位于 `node_modules/dotenv-expand` 下，这里就没啥好说的，就是将几个环境变量的值放到一起

接下来让我们回到开头说的 `只有以 VUE_APP_ 开头的变量会被 webpack.DefinePlugin 静态嵌入到客户端侧的包中`，这个是在那里配置的呢？

我们知道 `vue-cli` 会对 `webpack` 进行配置扩展，所以我们发现在 `cli-service/lib/config/app.js` 中

```js
const resolveClientEnv = require('../util/resolveClientEnv')
    webpackConfig
      .plugin('define')
        .use(require('webpack/lib/DefinePlugin'), [
          resolveClientEnv(options)
        ])
```

我们接着找到 `resolveClientEnv` 方法，可以看到是在这里定义的，并且会将非 `VUE_APP_` 开头的过滤掉

```js
const prefixRE = /^VUE_APP_/

module.exports = function resolveClientEnv (options, raw) {
  const env = {}
  Object.keys(process.env).forEach(key => {
    if (prefixRE.test(key) || key === 'NODE_ENV') {
      env[key] = process.env[key]
    }
  })
  env.BASE_URL = options.publicPath

  if (raw) {
    return env
  }

  for (const key in env) {
    env[key] = JSON.stringify(env[key])
  }
  return {
    'process.env': env
  }
}
```

### 总结

1. `.env` 和 `.env.local` 定义的环境变量会被全局引用，并会与其它环境变量合并
2. 声明环境变量必须以 `VUE_APP_` 开头，不然会被过滤掉
3. 除了 `VUE_APP_*` 变量之外，在你的应用代码中始终可用的还有两个特殊的变量：`NODE_ENV` 和 `BASE_URL`

最后，谢谢大家的观看，欢迎大家交流分享自己的开发心得~