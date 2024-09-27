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
  <div
    class="w-80 bg-gradient-to-br from-blue-100 to-purple-100 flex flex-col items-center justify-center min-h-screen p-6"
  >
    <div class="bg-white p-5 rounded-lg shadow-lg max-w-lg text-center mb-6">
      <h1 class="text-xl font-bold mb-3 text-blue-600">插件介绍</h1>
      <p class="text-gray-700 text-sm text-justify">这个插件可以让你任意修改网页内容。</p>
    </div>
    <!-- 更新开关组件 -->
    <div class="flex items-center space-x-4 bg-white p-4 rounded-lg shadow-md">
      <span class="text-gray-700 font-medium">编辑模式</span>
      <label class="switch">
        <input type="checkbox" v-model="designModeEnabled" @change="toggleDesignMode" />
        <span class="slider round"></span>
      </label>
    </div>
  </div>
</template>

<style scoped>
/* 开关样式更新 */
.switch {
  @apply relative inline-block w-14 h-8;
}

.switch input {
  @apply opacity-0 w-0 h-0;
}

.slider {
  @apply absolute cursor-pointer inset-0 bg-gray-300 transition-all duration-300 ease-in-out rounded-full;
}

.slider:before {
  @apply absolute content-[''] h-6 w-6 left-1 bottom-1 bg-white transition-all duration-300 ease-in-out rounded-full;
}

input:checked + .slider {
  @apply bg-blue-500;
}

input:checked + .slider:before {
  @apply transform translate-x-6;
}

input:focus + .slider {
  @apply ring-2 ring-blue-300;
}
</style>
