---
title: "Vue3 + AI 实战：从零搭建智能对话小助手"
date: 2026-04-20
tags: ["Vue3", "人工智能"]
category: "前端"
source: "https://blog.csdn.net/Zmt01015621/article/details/160345647"
---

## 前言：

最近陆续吃透了 Vue3 的核心用法，又初步接触了 AI 接口调用相关知识，一直想把两者结合起来，找一个简单易落地的小项目来巩固所学——既夯实 Vue3 的实操能力，也能熟悉 AI 与前端的联动逻辑。思来想去，AI 对话小助手是最适合新手的入门场景，于是结合所学知识，整理了一套从零到一的实现思路，同时梳理了新手学习过程中可能踩的坑和实用技巧。今天这篇博客，就把这份完整的落地指南分享出来，适合和我一样刚入门 Vue3 + AI、还未实际编写过相关项目的前端新手，全程无复杂配置，跟着步骤就能一步步搭建出属于自己的第一个 AI 前端项目～

## 一、技术栈说明

本次分享的项目没有选用复杂的技术栈，全部是前端新手容易上手的工具，重点聚焦"Vue3 与 AI 接口的结合"，具体如下：

- **核心框架：** Vue3（使用 setup 语法糖，简洁高效，告别 Options API 的繁琐）
- **构建工具：** Vite（比 Webpack 启动更快，配置更简单，适合快速开发小项目）
- **请求工具：** Axios（处理 HTTP 请求，用于调用 AI 接口）
- **状态管理：** Pinia（可选，用于管理聊天记录和加载状态，比 Vuex 更简洁）
- **AI 能力：** 通用大模型 API（无需深入 AI 算法，调用接口即可实现智能对话）

**提示：** 不用纠结于技术栈的"高大上"，能落地、能巩固知识点，就是最适合新手的选择。

## 二、项目核心思路（3步走，逻辑清晰不绕弯）

整个项目的核心逻辑非常简单，不用复杂的架构设计，重点实现"输入-请求-渲染"的完整流程，具体分为3步，适合新手逐步理解和落地：

1. **搭建 Vue3 基础页面：** 设计简单的聊天气泡布局，包含用户输入框、发送按钮、聊天记录展示区，保证样式简洁清爽，重点练习 Vue3 基础布局和指令使用。
2. **调用 AI 接口：** 通过 Axios 封装请求函数，将用户输入的内容传递给 AI 接口，学习处理请求参数和响应数据的方法，掌握前端调用 AI 接口的核心逻辑。
3. **响应式渲染：** 利用 Vue3 的 ref、watch 等响应式特性，将 AI 返回的回答实时渲染到页面上，同时处理加载状态、错误提示，提升用户体验，巩固 Vue3 响应式相关知识点。

简单来说，就是"前端页面收集用户输入 → 调用 AI 接口获取回答 → 实时渲染到页面"，全程无复杂逻辑，新手可轻松跟上思路，逐步落地实现。

## 三、关键代码片段

这部分是博客的核心，整理了项目各环节的关键代码，每个片段都标注了用途和注释，避免冗余，新手可以直接复制到自己的项目中使用，重点关注 Vue3 语法和 AI 接口调用的结合，快速上手实操。

### 1. 页面结构（Chat.vue）

**核心：** 实现聊天气泡布局，绑定输入值和点击事件，用 v-for 渲染聊天记录，添加加载状态提示，重点练习 Vue3 模板语法和指令使用。

```vue
<template>
  <div class="chat-container">
    <!-- 聊天记录展示区 -->
    <div class="chat-history">
      <div
        v-for="(item, index) in chatList"
        :key="index"
        :class="['chat-item', item.role === 'user' ? 'user' : 'ai']"
      >
        <div class="chat-avatar">{{ item.role === 'user' ? '我' : 'AI' }}</div>
        <div class="chat-content">{{ item.content }}</div>
      </div>
      <!-- 加载状态提示 -->
      <div class="loading" v-if="loading">AI 正在思考...</div>
    </div>

    <!-- 输入框和发送按钮 -->
    <div class="chat-input">
      <input
        v-model="inputValue"
        placeholder="请输入你的问题..."
        @keyup.enter="sendMessage"
        :disabled="loading"
      />
      <button @click="sendMessage" :disabled="loading || !inputValue">发送</button>
    </div>
  </div>
</template>
```

### 2. 核心逻辑（setup 语法糖）

**核心：** 定义响应式数据，封装 AI 接口请求函数，处理发送消息、渲染聊天记录的逻辑，包含加载状态和简单的错误处理，重点练习 Vue3 setup 语法和异步请求使用。

```vue
<script setup>
import { ref } from 'vue';
import { requestAI } from '@/utils/aiRequest'; // 导入封装好的 AI 接口请求函数

// 响应式数据：聊天记录、输入框内容、加载状态
const chatList = ref([
  { role: 'ai', content: '你好～我是你的 AI 对话助手，有什么问题可以问我哦！' }
]);
const inputValue = ref('');
const loading = ref(false);

// 发送消息函数
const sendMessage = async () => {
  // 避免空消息发送
  if (!inputValue.value.trim()) return;

  // 先将用户消息添加到聊天记录
  chatList.value.push({
    role: 'user',
    content: inputValue.value.trim()
  });
  // 清空输入框
  inputValue.value = '';
  // 开启加载状态
  loading.value = true;

  try {
    // 调用 AI 接口，获取回答
    const res = await requestAI(chatList.value);
    // 将 AI 回答添加到聊天记录
    chatList.value.push({
      role: 'ai',
      content: res.data.content
    });
  } catch (err) {
    // 错误处理，提示用户
    chatList.value.push({
      role: 'ai',
      content: '抱歉，出现错误，请稍后再试～'
    });
    console.log('AI 接口请求失败：', err);
  } finally {
    // 关闭加载状态
    loading.value = false;
  }
};
</script>
```

### 3. AI 接口封装（utils/aiRequest.js）

**核心：** 统一封装 AI 接口请求，处理请求头、请求参数，解决跨域问题（重点！新手必看），学习 Axios 封装和接口调用的最佳实践。

```javascript
import axios from 'axios';

// 创建 Axios 实例
const service = axios.create({
  baseURL: '/api', // 本地代理地址，解决跨域（关键配置）
  timeout: 30000, // 超时时间，AI 接口响应可能较慢，适当设置长一点
  headers: {
    'Content-Type': 'application/json',
    // 这里替换成自己的 AI 接口密钥（根据具体 AI 平台要求配置）
    'Authorization': `Bearer ${import.meta.env.VITE_AI_API_KEY}`
  }
});

// 封装 AI 请求函数
export const requestAI = async (chatList) => {
  return await service({
    method: 'post',
    url: '/chat/completions', // 具体 AI 接口地址，根据平台文档调整
    data: {
      model: 'gpt-3.5-turbo', // 模型版本，根据自己的需求调整
      messages: chatList, // 聊天记录，传递给 AI，实现多轮对话
      temperature: 0.7 // 回答随机性，0-1 之间，值越小越严谨
    }
  });
};
```

### 4. 跨域配置（vite.config.js）

**核心：** 新手最容易踩的坑——跨域报错，通过 Vite 代理配置解决，直接复制到自己的 vite.config.js 中即可，重点掌握前端跨域的基础解决方法。

```javascript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  // 跨域代理配置
  server: {
    proxy: {
      '/api': {
        target: 'https://api.openai.com', // 替换成自己的 AI 接口基础地址
        changeOrigin: true, // 允许跨域
        rewrite: (path) => path.replace(/^\/api/, '') // 重写路径，去掉 /api 前缀
      }
    }
  }
});
```

### 5. 简单样式（Chat.vue）

**核心：** 简洁清爽的聊天气泡样式，不用复杂设计，保证可读性即可，可根据自己的喜好调整颜色和布局，练习 Vue3 组件样式的编写。

```vue
<style scoped>
.chat-container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 20px;
  box-sizing: border-box;
}

.chat-history {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 8px;
}

.chat-item {
  display: flex;
  margin-bottom: 15px;
  max-width: 70%;
}

.user {
  flex-direction: row-reverse;
  margin-left: auto;
}

.chat-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #42b983;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 10px;
  flex-shrink: 0;
}

.user .chat-avatar {
  background-color: #2c3e50;
  margin-right: 0;
  margin-left: 10px;
}

.chat-content {
  padding: 10px 15px;
  border-radius: 18px;
  background-color: #f5f5f5;
  line-height: 1.5;
}

.user .chat-content {
  background-color: #42b983;
  color: white;
}

.chat-input {
  display: flex;
  gap: 10px;
}

.chat-input input {
  flex: 1;
  padding: 12px 15px;
  border: 1px solid #eee;
  border-radius: 24px;
  outline: none;
  font-size: 16px;
}

.chat-input button {
  padding: 0 20px;
  border: none;
  border-radius: 24px;
  background-color: #42b983;
  color: white;
  font-size: 16px;
  cursor: pointer;
}

.chat-input button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.loading {
  text-align: center;
  color: #999;
  font-size: 14px;
  margin: 10px 0;
}
</style>
```

## 四、新手避坑指南（重点！少走弯路）

这部分是结合新手学习特点，整理的常见问题和解决方法——这些都是新手尝试搭建该项目时，最容易遇到的坑，提前了解能节省大量学习时间，避免踩坑内耗。

| 坑 | 问题描述 | 解决方法 |
|---|---------|---------|
| 坑1 | 调用 AI 接口时出现"跨域"报错 | 配置 Vite 代理（如上面的 vite.config.js 配置），通过本地代理转发请求，避免浏览器跨域限制。注意：替换 target 为自己的 AI 接口基础地址，不要直接写 AI 接口地址在代码中。 |
| 坑2 | AI 接口响应过慢，页面没有加载状态 | 用 ref 定义 loading 状态，请求前设为 true，请求结束（成功/失败）后设为 false，同时在页面上添加加载提示，提升用户体验，避免用户重复点击发送。 |
| 坑3 | Vue3 中调用接口后，聊天记录没有实时渲染 | 确保聊天记录 chatList 是用 ref 定义的响应式数据，push 数据后，Vue3 会自动检测到变化，无需手动刷新页面；如果还是不渲染，检查是否有语法错误，或重启 Vite 服务。 |
| 坑4 | AI 接口密钥泄露 | 将密钥放在 .env 文件中，通过 import.meta.env 读取，不要直接写在代码中（尤其是上传到 GitHub 等平台时），避免密钥泄露导致接口被滥用。 |
| 坑5 | 输入空消息时，按钮可以点击 | 在 sendMessage 函数中添加判断，过滤空消息（inputValue.value.trim()），同时给按钮添加 disabled 状态，当输入框为空或加载中时，禁用按钮。 |

## 五、项目效果展示（预期效果）

按照上述步骤搭建完成后，AI 对话小助手的预期效果简洁实用，具体如下，可作为搭建过程中的参考：

- **页面布局：** 清晰的聊天气泡，用户消息居右、AI 消息居左，加载状态提示明显，输入框和发送按钮布局合理，适配不同屏幕尺寸。
- **核心功能：** 支持用户输入问题、点击发送或回车发送，AI 实时响应并渲染回答，支持多轮对话，错误时给出清晰提示，流程顺畅。
- **用户体验：** 加载状态反馈及时，按钮禁用逻辑合理，聊天气泡样式清爽，可读性强，符合新手项目的预期效果。

（提示：如果后续实际搭建完成，可在这里插入项目截图，比如页面整体截图、聊天效果截图，会更直观；未搭建完成时，用文字描述预期效果即可。）

## 六、学习总结与后续计划

结合 Vue3 和 AI 基础知识点，整理完这套 AI 对话小助手的搭建思路后，我对前端与 AI 的联动有了更清晰的认知——不仅巩固了 Vue3 的核心知识点（setup 语法糖、ref 响应式、组件布局），还初步掌握了 AI 接口的调用、Axios 封装、跨域处理等实用技巧，也理解了"前端+AI"的入门逻辑。

其实前端结合 AI 并没有想象中那么难，新手入门的关键不是深入 AI 算法，而是掌握"前端页面与 AI 接口的联动"，先把简单的场景落地，再逐步优化升级。对于还未实际编写过相关项目的我来说，这套思路也是后续实操的重要参考，能帮助自己快速上手，少走弯路。

**后续计划：** 按照这份思路，实际动手搭建该 AI 对话小助手项目，完成后进一步优化功能，比如添加历史聊天记录保存（localStorage 实现）、深色模式切换、聊天记录清空功能；同时深入学习 AI 流式输出，实现像 ChatGPT 一样的打字机效果，尝试接入更多 AI 能力（比如文本总结、翻译），持续提升自己的前端+AI 实战能力。

最后，希望这篇博客能帮到和我一样刚入门 Vue3 + AI、还未实际编写过相关项目的前端新手，大家一起学习、一起进步，早日将所学知识落地，实现自己的技术目标～
