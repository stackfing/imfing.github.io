---
title: 手摸手，带你用 vue 动画实现原生 app 切换效果，丝滑般的体验
date: 2018-05-22 13:07:59
tags:
---
先来看效果图

![效果](http://upload-images.jianshu.io/upload_images/4438070-f40294da0943c89b?imageMogr2/auto-orient/strip)

完整源码在 github 中：

https://github.com/imfing/vuexlearn

## 准备
开始之前您需要有 vue 基础，以及安装好 vue-cli

## 开始
新建 vue 项目：`vue init webpack vuexlearn`
记住安装的时候需要选择 vue-router

进入 vuexlearn 目录之后安装 vuex：
这里使用 npm 安装 `npm install vuex --save` 您也可以使用其他方式安装，具体请参考 vuex 官方文档。

在安装好 vuex 之后，您就可以使用 `npm run dev` 命令运行您的 web 应用了。

现在在 `main.js` 文件中引入 vuex
### main.js

```
import vuex from 'vuex'
Vue.use(vuex);
```

在下方添加我们的 vuex 状态树
```
var store = new vuex.Store({//store对象
  state: {
    states: 'turn-on'
  },
  mutations: {
    setTransition(state, states) {
      state.states = states
    }
  }
})
```

 `state.states` 就是用来记录我们目前的切换状态， `turn-on` 为页面入栈，`turn-off` 是页面出栈。

`setTransition(state, states)` 方法用来设置 states 的值，在需要的时候我们会调用它。

接下来，我们新建一个 `common` 组件，作为我们的作标题栏
### common.vue
```
<template>
    <div class="header">
        <div class="left" @click="back">
            back
        </div>
        <div class="center">
            {{title}}
        </div>
    </div>
</template>

<script>
export default {
  data() {
    return {};
  },
  props: ["title"],
  methods: {
    back() {
      this.$store.commit("setTransition", "turn-off");
      this.$router.back(-1);
    }
  }
};
</script>

<style scoped>
.header {
  position: fixed;
  width: 100%;
  height: 40px;
  line-height: 40px;
  background-color: rgb(100, 231, 60);
}
.clearfix {
  overflow: auto;
}
.left {
  position: fixed;
  left: 0px;
  width: 60px;
}
.center {
  left: 50%;
  position: fixed;
}
</style>

```

这里通过 `props` 拿到 `name` 的值，渲染在标题栏上

这里的切换核心就是在点击返回的时候，设置整个页面的动画效果

新建 4 个页面，其他的页面雷同，所以这里只贴出一个页面
### A.vue

```
<template>
    <div class="A">
        <common title="a"></common>
        <div class="bottom">
            bottom
        </div>
    </div>
</template>

<script>
import common from "./common";
export default {
  data() {
    return {};
  },
  components: {
    common
  }
};
</script>

<style scoped>
.A {
  width: 100%;
  height: 100%;
  background-color: rgb(214, 198, 52);
  position: fixed;
}

.bottom {
  width: 100%;
  height: 50px;
  background-color: red;
  position: fixed;
  bottom: 0px;
}
</style>

```

#### App.vue
```
<template>
  <div id="app">
    <router-link to="/A" @click.native="clickLink">A</router-link>
    <router-link to="/B" @click.native="clickLink">B</router-link>
    <router-link to="/C" @click.native="clickLink">C</router-link>
    <router-link to="/D" @click.native="clickLink">D</router-link>
    <transition :name="$store.state.states">
      <router-view/>
    </transition>
    <div>Index Page</div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
    };
  },
  methods: {
    clickLink() {
      this.$store.commit("setTransition", "turn-on");
    }
  },
  mounted() {
    var _this = this;
    window.addEventListener(
      "popstate",
      function(e) {
        _this.$store.commit("setTransition", "turn-off");
      },
      false
    );
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
.btn {
  width: 50%;
}
html,
body,
#app {
  height: 100%;
}
.turn-on-enter {
  transform: translate3d(100%, 0, 0);
}
.turn-on-leave-to {
  /* transform: translate3d(-20%, 0, 0); */
}
.turn-on-enter-active,
.turn-on-leave-active {
  transition: transform 0.4s ease;
}
.turn-off-enter {
  /* transform: translate3d(-20%, 0, 0); */
}
.turn-off-leave-to {
  transform: translate3d(100%, 0, 0);
}
.turn-off-enter-active,
.turn-off-leave-active {
  transition: transform 0.4s ease;
}
.turn-off-leave-active {
  z-index: 2;
}
</style>

```

切换效果就在这里定义了，通过 vuex 全局保存变量达到页面入栈、出栈的动画效果。

完整源码在 github 中：

https://github.com/imfing/vuexlearn

最后在看一下效果图：

![效果](http://upload-images.jianshu.io/upload_images/4438070-a4e3beb7c1befb3d?imageMogr2/auto-orient/strip)
