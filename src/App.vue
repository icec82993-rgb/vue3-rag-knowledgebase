<template>
  <div style="padding: 24px; max-width: 800px; margin: 0 auto; font-family: sans-serif;">
    <h2 style="color: #2c3e50; border-bottom: 2px solid #42b983; padding-bottom: 10px;">
      🔍 RAG 私有知识库匹配与 Prompt 编排看板
    </h2>
    
    <div style="margin-top: 20px;">
      <label style="font-weight: bold; display: block; margin-bottom: 8px;">1. 输入用户提问 (User Query):</label>
      <input 
        v-model="userQuery" 
        placeholder="请输入您的问题，例如：组件通信..." 
        style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box;"
      />
    </div>

    <div style="margin-top: 24px;">
      <p style="font-weight: bold; margin-bottom: 8px;">⚡ 向量数据库模拟检索切片 (Matched Chunks):</p>
      <div v-for="(chunk, index) in matchedChunks" :key="index" style="background: #f0f9eb; border: 1px solid #c2e7b0; padding: 12px; margin-bottom: 10px; border-radius: 4px;">
        <div style="display: flex; justify-content: space-between; font-size: 13px; color: #67c23a; margin-bottom: 6px;">
          <span>📄 切片 #{{ index + 1 }}</span>
          <span>相关度相似得分: <strong>{{ chunk.score }}</strong></span>
        </div>
        <p style="margin: 0; color: #303133; font-size: 14px;">{{ chunk.content }}</p>
      </div>
    </div>

    <div style="margin-top: 24px;">
      <label style="font-weight: bold; display: block; margin-bottom: 8px;">2. 动态组装给大模型的 Prompt 模板:</label>
      <textarea 
        :value="assembledPrompt" 
        readonly
        rows="12" 
        style="width: 100%; padding: 12px; font-family: monospace; background: #f4f4f5; border: 1px solid #dcdfe6; border-radius: 4px; box-sizing: border-box; color: #606266;"
      ></textarea>
    </div>

    <button style="margin-top: 16px; width: 100%; padding: 12px; background: #409eff; color: white; border: none; border-radius: 4px; font-size: 16px; cursor: pointer; font-weight: bold;">
      将增强 Prompt 送入大模型
    </button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const userQuery = ref('组件通信')

const mockVectorDatabase = [
  { keywords: ['传参', 'props', 'emit', '组件通信'], content: 'Vue 3 中父传子使用 defineProps，子传父使用 defineEmits。', score: '0.9574' },
  { keywords: ['provide', 'inject', 'pinia', '跨组件'], content: '跨组件全局通信可以使用 Provide / Inject，或者引入 Pinia 状态管理库。', score: '0.8869' },
  { keywords: ['v-model', '双向绑定'], content: 'Vue 3 的 v-model 默认编译为 modelValue 属性和 update:modelValue 事件。', score: '0.4120' }
]

const matchedChunks = computed(() => {
  if (!userQuery.value.trim()) return []
  return mockVectorDatabase.filter(chunk => 
    chunk.keywords.some(keyword => userQuery.value.toLowerCase().includes(keyword.toLowerCase()))
  )
})

const assembledPrompt = computed(() => {
  const systemRole = '【系统角色说明】你是一个严谨的知识库助手。请严格基于以下给定的已知信息回答用户问题，如果已知信息中没有提到，请委婉拒绝，不要胡乱编造。\n'
  const divider = '-----------------------\n'
  
  let context = '已知参考资料：\n'
  if (matchedChunks.value.length > 0) {
    matchedChunks.value.forEach((chunk, i) => {
      context += `[资料${i + 1}] ${chunk.content}\n`
    })
  } else {
    context += '（未检索到匹配的本地私有知识切片）\n'
  }

  const querySection = `\n用户真实问题：${userQuery.value}\n`
  const tail = `\n请基于上述小抄，进行准确回答：`

  return `${systemRole}${divider}${context}${divider}${querySection}${divider}${tail}`
})
</script>