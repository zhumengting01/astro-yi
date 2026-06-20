---
title: "Vue3 + Pinia 状态管理实战教程：从入门到模块化+持久化，彻底告别 Vuex"
date: 2026-04-11
tags: ["Vue3", "Pinia"]
category: "前端"
source: "https://blog.csdn.net/Zmt01015621/article/details/160047016"
---

> 本文适合：Vue3 初学者、从 Vue2 迁移 Vue3 的开发者、想要替代 Vuex 的前端同学
> 
> 开发环境：Vue3.4 + Pinia 2.1 + Vite
> 
> 来源：https://blog.csdn.net/Zmt01015621/article/details/160047016

## 前言

在 Vue3 成为默认版本后，状态管理方案也迎来了大更新。曾经的 Vuex 已经逐渐被 Pinia 取代，它不仅是 Vue 官方推荐的新一代状态管理库，还拥有更简洁的 API、更好的 TypeScript 支持、更轻量的体积，以及开箱即用的模块化能力。

很多同学还停留在 Vuex 的思维里，觉得状态管理很复杂。其实 Pinia 上手非常简单，今天这篇文章就带你从零学会 Pinia，包含：

**安装 → 定义 Store → 组件使用 → 模块化 → 模块间调用 → 数据持久化 → 高频踩坑。**

学完这一篇，基本可以直接在项目里替代 Vuex。

## 一、为什么 Pinia 能替代 Vuex？

| 优势 | 说明 |
|------|------|
| API 更简洁 | 移除了 Vuex 中的 mutation，同步异步都写在 actions 里，代码更少更清晰。 |
| TypeScript 支持更好 | 类型推导天然完善，不用额外配置，写代码提示非常友好。 |
| 模块化更简单 | 不需要配置 modules 和 namespaced，新建文件就是一个模块。 |
| 异步逻辑更直观 | actions 直接支持 async/await，不用再写繁琐的 dispatch。 |
| 体积更小、性能更好 | 打包后几乎不占体积，基于 Vue3 响应式系统，更稳定。 |
| 官方亲儿子，长期维护 | 已经纳入 Vue 官方生态，未来只会越来越主流。 |

## 二、安装 Pinia

在你的 Vue3 项目根目录执行安装命令：

```bash
# npm
npm install pinia

# yarn
yarn add pinia

# pnpm（推荐）
pnpm add pinia
```

## 三、全局注册 Pinia

打开项目入口文件 `src/main.js`，全局注册 Pinia 实例：

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia'

const app = createApp(App)
// 注册 Pinia
app.use(createPinia())
app.mount('#app')
```

注册完成后，整个项目所有组件都可以使用 Pinia 管理状态。

## 四、定义第一个 Store

建议在 src 下新建文件夹 stores，统一管理所有状态模块。

**目录结构：**
```
src
└── stores
    └── user.js  # 用户信息模块
```

**编写 user.js：**

```javascript
import { defineStore } from 'pinia'

// 第一个参数是 store 的唯一 ID，不能重复
export const useUserStore = defineStore('user', {
  // 存储全局状态，必须是函数形式
  state: () => {
    return {
      token: '',
      userInfo: null,
      age: 18,
      isLogin: false
    }
  },

  // 计算属性，有缓存效果
  getters: {
    doubleAge(state) {
      return state.age * 2
    },
    isAdult(state) {
      return state.age >= 18
    }
  },

  // 操作状态的方法，同步异步都可以写
  actions: {
    changeAge(num) {
      this.age += num
    },

    // 模拟登录请求
    async login(userForm) {
      // 实际项目替换为接口请求
      const mockToken = 'TOKEN_' + Date.now()
      this.token = mockToken
      this.userInfo = userForm
      this.isLogin = true
    },

    logout() {
      this.token = ''
      this.userInfo = null
      this.isLogin = false
      this.age = 18
    }
  }
})
```

**要点说明：**

- `state` 必须是函数，避免多组件实例数据污染。
- `getters` 类似组件中的 computed，依赖不变时会缓存。
- `actions` 支持 async/await，直接用 this 修改状态。

## 五、在组件中使用 Pinia

新建一个 Vue 组件，演示如何使用 Store 中的状态和方法：

```vue
<template>
  <div class="demo-box">
    <h3>Pinia 基础使用演示</h3>
    <p>登录状态：{{ isLogin ? '已登录' : '未登录' }}</p>
    <p>年龄：{{ age }}</p>
    <p>双倍年龄：{{ doubleAge }}</p>

    <button @click="changeAge(1)">年龄 +1</button>
    <button @click="handleLogin">模拟登录</button>
    <button @click="logout">退出登录</button>
  </div>
</template>

<script setup>
import { useUserStore } from '@/stores/user'
import { storeToRefs } from 'pinia'

const userStore = useUserStore()

// 解构 state 必须用 storeToRefs 保持响应式
const { age, isLogin, doubleAge } = storeToRefs(userStore)
// 方法可以直接解构
const { changeAge, logout } = userStore

const handleLogin = () => {
  userStore.login({
    username: 'CSDN开发者',
    role: '前端工程师'
  })
}
</script>

<style scoped>
.demo-box {
  padding: 20px;
}
button {
  margin: 0 10px;
  padding: 4px 12px;
  cursor: pointer;
}
</style>
```

### 重要注意事项

**直接解构会丢失响应式：**

```javascript
// ❌ 错误写法
const { age } = userStore

// ✅ 正确写法
const { age } = storeToRefs(userStore)
```

## 六、Pinia 模块化与模块间调用

大型项目必须拆分模块，Pinia 模块化非常简单：多建几个 store 文件即可。

**示例目录：**
```
stores
├── user.js    # 用户模块
└── cart.js    # 购物车模块
```

**编写 cart.js：**

```javascript
import { defineStore } from 'pinia'

export const useCartStore = defineStore('cart', {
  state: () => ({
    goodsList: [],
    totalPrice: 0
  }),
  actions: {
    addGoods(item) {
      this.goodsList.push(item)
      this.totalPrice += item.price
    }
  }
})
```

### 在一个模块中调用另一个模块

在 user.js 的 actions 中引入购物车模块，实现业务联动：

```javascript
import { useCartStore } from './cart'

actions: {
  buyGoods(item) {
    if (!this.isLogin) {
      alert('请先登录')
      return
    }
    const cartStore = useCartStore()
    cartStore.addGoods(item)
  }
}
```

**组件中使用：**

```javascript
const buy = () => {
  userStore.buyGoods({ name: 'Vue3实战课程', price: 199 })
}
```

这样就实现了登录判断 + 加入购物车的真实业务逻辑。

## 七、Pinia 数据持久化（刷新页面不丢失）

Pinia 默认数据存在内存中，刷新会重置。实际项目中 token、用户信息必须持久化。

**使用插件：** `pinia-plugin-persistedstate`

### 1. 安装插件

```bash
pnpm add pinia-plugin-persistedstate
```

### 2. 在 main.js 中注册

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const app = createApp(App)
const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

app.use(pinia)
app.mount('#app')
```

### 3. 给 Store 开启持久化

```javascript
export const useUserStore = defineStore('user', {
  state: () => ({ ... }),
  getters: {},
  actions: {},
  // 开启持久化
  persist: true
})
```

### 4. 高级配置（推荐）

```javascript
persist: {
  key: 'MY_APP_USER',        // 自定义 storage 键名
  storage: sessionStorage,   // 可选 localStorage / sessionStorage
  paths: ['token', 'userInfo'] // 只持久化指定字段
}
```

## 八、Pinia 常用实用技巧

### 1. 重置状态到初始值

```javascript
userStore.$reset()
```

### 2. 批量修改状态

```javascript
userStore.$patch({
  age: 25,
  isLogin: true
})
```

### 3. 监听状态变化

```javascript
userStore.$subscribe((mutation, state) => {
  console.log('用户状态发生变化', state)
})
```

## 九、高频踩坑总结（新手必看）

| 坑 | 说明 |
|---|------|
| 直接解构 state 导致响应式丢失 | 必须使用 storeToRefs |
| state 写成对象而不是函数 | 会造成多组件数据污染、状态错乱 |
| actions 中使用箭头函数 | 会导致 this 指向错误，无法修改 state |
| 多个 store 的 id 重复 | 会造成状态覆盖、页面数据异常 |
| 持久化后 Date 类型变成字符串 | 需手动转换回 Date 对象 |
| 模块间循环引用 | A 引入 B，B 引入 A，会导致运行报错 |

## 十、总结

- Pinia 凭借简洁 API、优秀 TS 支持、开箱即用的模块化，已经完全替代 Vuex。
- **核心结构：** state 存数据、getters 做计算、actions 写逻辑。
- **模块化**无需配置，多文件拆分即可，模块间调用非常简单。
- **持久化**只需一个插件，轻松解决刷新丢失问题。
- 配合 Vue3 setup 语法糖，代码更简洁、维护成本更低。

无论是个人项目、后台管理系统还是企业级应用，Pinia 都是目前 Vue3 状态管理的最佳选择。
