在做 nuxt 项目中，页面缓存的问题一直是比较头疼的一件事，这个问题我 google 好久也并没有收获，几乎都是很粗暴的告诉你使用 keep-alive，也不管相关的坑。

本篇将把我在**nuxt**中**keep-alive**的使用心得分享，希望可以帮助你更优雅的去处理页面缓存问题。

---

## layouts 添加 keep-alive

- 首先建立[nuxt](https://nuxtjs.org/)项目
- 在`./layouts/default.vue`添加 keep-alive
  ```html
  <template>
    <div>
      <nuxt keep-alive />
    </div>
  </template>
  ```

---

## 处理 asyncData

因为加上`keep-alive`后`asyncData`依然会在路由切换的时候触发，规避这种问题我们做一个简单的处理

```js
// ./pages/index.vue
export default {
  async asyncData({ app, store, error }) {
    if (!process.server) return
    const params = {
      pageNum: 1
    }
    const { data } = await app.$getPostList(params)
    return { postList: data.result, pageNum: 2 }
  },
  data() {
    return {
      pageNum: 1,
      postList: []
    }
  },
  mounted() {
    if (this.$store.state.is_browser_first) {
      this.$store.commit('saveBrowserFirst', false)
    } else {
      const params = {
      pageNum: this.pageNum
    }
    const { data } = await app.$getPostList(params)
    this.postList = data.result
    }
  }
}
```
解释一下这段js中需要注意的点
- asyncData
  + `if (!process.server) return` 如果非服务端环境直接retuen。将**asyncData**只控制在服务端环境触发能为我们省去不少麻烦的事情。
- data
  + 在data中还要定义一次属性是因为如果不定义，在浏览器环境A页面进入B页面，B页面会报错属性不存在，原因是 asyncData 没触发。
- mounted
  + `if (this.$store.state.is_browser_first){}`,`is_browser_first`是我在vuex(./store/index.js)中定义的，默认值为`true`,主要目的是记录是否是第一次进入浏览器，如果是是就将`is_browser_first`设为`false`,也就是`saveBrowserFirst`方法。

---

## vuex 
```js
// ./store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
const store = () =>
  new Vuex.Store({
    state: {
      is_browser_first: true
    },
    mutations: {
      saveBrowserFirst(state, type) {
        state.is_browser_first = type
      }
    }
  })

export default store
```
---  

## 现存的问题
- error page
  + 当我们通过`error({ statusCode: 404 })`方法进入错误页面后，如果通过路由返回，再进入同等级路由页面时，页面还是错误页面样式。我的做法是不通过路由返回，通过`location`方法。

如果你有更好的解决方案欢迎与我交流，希望这篇可以帮助到为此事而头痛的coder。  