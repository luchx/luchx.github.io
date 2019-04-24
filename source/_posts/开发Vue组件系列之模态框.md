---
title: 开发Vue组件系列之模态框
date: 2019-04-18 23:02:18
categories: 系列教程
tags: [vue, 组件, 模态框] 
top: true
---

> 开发Vue组件系列之模态框,主要有标题、内容、定时器、按钮文案、按钮事件回调、遮罩层这些可配置项。本次开发得组件是本系列的第一个组件,后期也会有更多系列教程推出。

### 使用命令行安装或下载
```bash
npm install lcxModal -S
```

### 引入方式
- import lcxModal from 'lcxModal';
- const lcxModal = require('lcxModal');
- <script src="xxx/lcxModal.js"></script>

### 使用
- Vue.use(lcxModal); 
- Vue.component('lcxModal', lcxModal); 

### 项目结构
```bash
├── src                            # 项目源码。开发的时候代码写在这里。
│   ├── lib                 # 组件目录
|   |   |--lcxModal               # 模态框组件
│   ├── App.vue                    # 项目根视图
│   ├── main.js                    # 程序主入口
```

### 部分截图

<div align="center">
    <img src="data:image/png;base64,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" width="300" alt="默认样式" />
    <img src="data:image/png;base64,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" width="300" alt="素样式" />
    <img src="data:image/png;base64,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" width="300" alt="自定义样式" />
</div>

### modal组件模板

> 使用 `transition` 可以为组件添加动效;对应的组件模板内容如下

```html
<template>
  <transition name="toggle">
    <section class="modal" v-show="visible">
      <div class="modal-mask" v-show="showMask" @click="clickMask"></div>
      <section class="modal-content modal-center" :style="contentStyle">
        <header class="modal-header" :class="{ 'modal-plain': plain }" v-if="showHeader">
          <slot name="header">{{ title }}</slot>
          <span class="closed" v-if="showClose" @click.stop="onClose">
            关闭
          </span>
        </header>
        <main class="modal-body">
          <slot>
            <div class="text-tips">{{ text }}</div>
          </slot>
        </main>
        <footer class="modal-footer" v-if="showFooter">
          <slot name="footer">
            <button class="modal-btn modal-btn-primary" @click.stop="onConfirm">
              {{ confirmBtnText }}
            </button>
            <button class="modal-btn modal-btn-default" @click.stop="onClose">
              {{ cancelBtnText }}
            </button>
          </slot>
        </footer>
      </section>
    </section>
  </transition>
</template>
```

### 添加组件属性及操作方法

> 添加组件的属性,其中`duration`属性如果设定的数值小于10,则会乘以1000;否则按传递的数值计算

```js
<script>
  export default {
    name: "EchiModal",
    props: {
      visible: {
        type: Boolean,
        default: false
      },
      title: {
        type: String,
        default: "标题"
      },
      text: {
        type: String,
        default: "提示信息"
      },
      tinyBar: {
        type: Boolean,
        default: false
      },
      confirmBtnText: {
        type: String,
        default: "确定"
      },
      cancelBtnText: {
        type: String,
        default: "返回"
      },
      contentStyle: {
        type: Object,
        default: () => {}
      },
      showClose: {
        type: Boolean,
        default: true
      },
      plain: {
        type: Boolean,
        default: false
      },
      showHeader: {
        type: Boolean,
        default: true
      },
      showFooter: {
        type: Boolean,
        default: true
      },
      showMask: {
        type: Boolean,
        default: true
      },
      onMask: {
        type: Boolean,
        default: false
      },
      duration: {
        type: Number,
        default: 0
      }
    },
    watch: {
      visible(nv) {
        if (nv) {
          this.closeTimerHandle()
        }
      }
    },
    data() {
      return {
        closeTimer: null,
      }
    },
    methods: {
      onClose() {
        this.$emit("on-close");
        this.hide();
      },
      onConfirm() {
        this.$emit("on-confirm");
      },
      hide() {
        this.$emit("update:visible", false);
        this.$emit("on-closed");
        clearTimeout(this.closeTimer);
        this.closeTimer = null;
      },
      clickMask() {
        if (this.onMask && this.showMask) {
          this.hide();
        }
      },
      closeTimerHandle() {
        try {
          if (this.duration <= 0) {
            return;
          }
          const duration = (this.duration < 10) ? (this.duration * 1000) : this.duration;
          clearTimeout(this.closeTimer);
          this.closeTimer = setTimeout(() => {
            this.onClose();
          }, duration);
        } catch (e) {
          console.log(e)
        }
      }
    }
  };
</script>
```

### 添加样式声明
```css
<style scoped lang="scss">
  *,
  :after,
  :before {
    box-sizing: border-box;
    outline: none;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  $color-tips: #1ab394;
  $color-text: rgba(255, 255, 255, 0.6);
  $color-info: #ff9900;
  $color-default: #ccc;

  .modal {
    display: block;
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 99;

    .modal-mask {
      display: block;
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      background-color: rgba(0, 0, 0, 0.5);
    }

    .modal-center {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }

    .modal-content {
      display: flex;
      flex-direction: column;
      min-width: 360px;
      box-shadow: 0 1px 8px 0 rgba(0, 30, 24, 0.05);
      background-color: #fff;
      border-radius: 6px;
      overflow: hidden;
    }

    .modal-header {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 44px;
      font-size: 18px;
      padding: 0 20px;
      font-weight: 500;
      color: #fff;
      background-color: $color-tips;
      z-index: 9999;

      .closed {
        position: absolute;
        top: 50%;
        right: 0;
        font-size: 12px;
        padding: 8px 16px;
        border-radius: 4px;
        cursor: pointer;
        color: #fff;
        transform: translateY(-50%);
      }

      &.modal-plain {
        background-color: rgba(245,
          245,
          245,
          1);
        color: $color-tips;

        .closed {
          color: $color-info;
        }
      }
    }

    .modal-body {
      display: block;
      flex: 1;
      background-color: #fff;
      overflow: hidden;
      overflow-y: auto;
      -webkit-overflow-scrolling: touch;
    }

    .modal-footer {
      display: block;
      width: 100%;
      padding: 20px 30px;
      text-align: center;
      background-color: #fff;

      .modal-btn {
        width: 80px;

        +.modal-btn {
          margin-left: 8px;
        }
      }
    }
  }

  .text-tips {
    display: block;
    width: 100%;
    font-size: 16px;
    text-align: center;
    color: #333;
    padding: 40px 60px;
  }

  .modal-btn {
    display: inline-flex;
    padding: 0 12;
    margin: 0;
    align-items: center;
    justify-content: center;
    font-size: 14px;
    font-weight: 400;
    height: 32px;
    text-align: center;
    white-space: nowrap;
    touch-action: manipulation;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    cursor: pointer;
    user-select: none;
    background-image: none;
    text-decoration: none;
    border: 1px solid transparent;
    border-radius: 4px;
    transition: all .3s ease;

    &:link,
    &:visited,
    &:hover,
    &:active {
      text-decoration: none;
    }
  }

  .modal-btn-default {
    background-color: $color-default;
    color: #fff;

    &:link {
      color: #fff;
      background-color: $color-default;
    }

    &:visited {
      color: #fff;
      background-color: $color-default;
    }

    &:hover {
      color: #fff;
      background-color: rgba(170, 170, 170, .85);
    }

    &:active {
      color: #fff;
      background-color: $color-default;
    }

    &[plain] {
      background-color: #fff;
      color: $color-default;
      border: 1px solid $color-default;
    }
  }

  .modal-btn-primary {
    background-color: $color-tips;
    color: #fff;

    &:link {
      color: #fff;
      background-color: $color-tips;
    }

    &:visited {
      color: #fff;
      background-color: $color-tips;
    }

    &:hover {
      color: #fff;
      background-color: rgba(26, 179, 148, 0.85);
    }

    &:active {
      color: #fff;
      background-color: $color-tips;
    }

    &[plain] {
      background-color: #fff;
      color: $color-tips;
      border: 1px solid $color-tips;
    }
  }

  .toggle-enter,
  .toggle-leave-active {
    opacity: 0;
    transform: translatey(-10px);
  }

  .toggle-enter-active,
  .toggle-leave-active {
    transition: all ease .2s;
  }
</style>
```

### 使用

```html
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <div>
      <button @click.stop="showModel_0 = true">
        显示默认样式
      </button>
      <button @click.stop="showModel_1 = true">
        显示素样式
      </button>
      <button @click.stop="showModel_2 = true">
        修改提示语
      </button>
      <button @click.stop="showModel_3 = true">
        自定义内容
      </button>
      <button @click.stop="showModel_4 = true">
        去除Footer
      </button>
      <button @click.stop="showModel_5 = true">
        去除Header
      </button>
      <button @click.stop="showModel_6 = true">
        设置3秒后自动关闭
      </button>
    </div>
    <EchiModal :visible.sync="showModel_0" title="显示默认样式"></EchiModal>
    <EchiModal :visible.sync="showModel_1" title="显示素样式" plain></EchiModal>
    <EchiModal :visible.sync="showModel_2" title="修改提示语" text="哈哈哈哈哈,我把提示信息修改了"></EchiModal>
    <EchiModal :visible.sync="showModel_3" title="自定义内容" :contentStyle="{width: '600px'}">
      <img alt="Vue logo" src="./assets/logo.png" />
    </EchiModal>
    <EchiModal :visible.sync="showModel_4" title="去除Footer" :showFooter="false"></EchiModal>
    <EchiModal :visible.sync="showModel_5" title="去除Header" :showHeader="false"></EchiModal>
    <EchiModal :visible.sync="showModel_6" title="设置3秒后自动关闭" :duration="3"></EchiModal>
  </div>
</template>

<script>
  import EchiModal from "./components/EchiModal.vue";

  export default {
    name: "app",
    components: {
      EchiModal
    },
    data() {
      return {
        showModel_0: false,
        showModel_1: false,
        showModel_2: false,
        showModel_3: false,
        showModel_4: false,
        showModel_5: false,
        showModel_6: false,
      }
    }
  };
</script>
```

> 感谢那您的观看,以上就是我为大家带来的模态框组件,本文同步更新于我的github[点击前往](https://github.com/Echi1993/echi-modal)。如果对您有帮助,请为我点个小星星