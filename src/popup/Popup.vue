<script setup lang="js">
import { ref, onMounted } from 'vue'

const designModeEnabled = ref(false)

onMounted(() => {
  // 获取设计模式状态
  chrome.storage.sync.get('designModeEnabled', (data) => {
    designModeEnabled.value = data.designModeEnabled
  })
})
const toggleDesignMode = () => {
  console.log('toggleDesignMode', designModeEnabled.value)
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    const currentTab = tabs[0]
    if (currentTab) {
      // 发送消息到内容脚本
      chrome.tabs.sendMessage(currentTab.id, {
        action: 'toggleDesignMode',
        enabled: designModeEnabled.value,
      })
      // 设置设计模式状态
      chrome.storage.sync.set({ designModeEnabled: designModeEnabled.value })
    }
  })
}
</script>

<template>
  <div class="w-80 bg-gray-100 flex flex-col items-center justify-center min-h-screen p-4">
    <div class="bg-white p-8 rounded shadow-md max-w-lg text-center mb-4">
      <h1 class="text-2xl font-bold mb-4">插件介绍</h1>
      <p class="text-gray-700 text-xl text-justify">这个插件可以让你任意修改网页内容。</p>
    </div>
    <!-- 添加开关组件 -->
    <div class="flex items-center mb-4">
      <span class="mr-2">编辑模式</span>
      <label class="switch">
        <input type="checkbox" v-model="designModeEnabled" @change="toggleDesignMode" />
        <span class="slider round"></span>
      </label>
    </div>
  </div>
</template>

<style scoped>
/* 开关样式 */
.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: 0.4s;
}

.slider:before {
  position: absolute;
  content: '';
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #2196f3;
}

input:checked + .slider:before {
  transform: translateX(26px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}
</style>
