<!-- src/content/Content.vue -->
<template></template>

<script setup>
import { ref } from 'vue'

console.log('Content script loaded', new Date().toISOString())

const anchorClickHandler = (e) => {
  e.preventDefault()
  console.log('Anchor click prevented')
}

const designModeEnabled = ref(false)

function disableAnchorDefaultBehavior() {
  console.log('Disabling anchor default behavior')
  document.querySelectorAll('a').forEach((a) => {
    a.addEventListener('click', anchorClickHandler)
  })
}

function enableAnchorDefaultBehavior() {
  console.log('Enabling anchor default behavior')
  document.querySelectorAll('a').forEach((a) => {
    a.removeEventListener('click', anchorClickHandler)
  })
}

function toggleDesignMode(enabled) {
  designModeEnabled.value = enabled
  document.designMode = enabled ? 'on' : 'off'
  console.log('Design mode set to:', document.designMode)

  if (enabled) {
    disableAnchorDefaultBehavior()
  } else {
    enableAnchorDefaultBehavior()
  }
}

chrome.storage.sync.get('designModeEnabled', (data) => {
  toggleDesignMode(data.designModeEnabled)
})

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  console.log('Message received in content script:', request)

  if (request.action === 'toggleDesignMode') {
    toggleDesignMode(request.enabled)
    sendResponse({ success: true, newState: document.designMode })
  } else if (request.action === 'getDesignModeStatus') {
    sendResponse({ enabled: designModeEnabled.value })
  } else {
    console.log('Unknown action:', request.action)
    sendResponse({ error: 'Unknown action' })
  }

  return true // 保持消息通道开放，以便异步发送响应
})

// 监听DOM变化，为新添加的链接添加事件监听器
const observer = new MutationObserver((mutations) => {
  if (designModeEnabled.value) {
    mutations.forEach((mutation) => {
      if (mutation.type === 'childList') {
        mutation.addedNodes.forEach((node) => {
          if (node.nodeType === Node.ELEMENT_NODE) {
            node.querySelectorAll('a').forEach((a) => {
              a.addEventListener('click', anchorClickHandler)
            })
          }
        })
      }
    })
  }
})

observer.observe(document.body, { childList: true, subtree: true })
</script>

<style></style>
