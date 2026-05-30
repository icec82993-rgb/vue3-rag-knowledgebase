<template>
  <div class="rag-wrapper">
    <header class="rag-header">
      <div class="logo">🔍 RAG-KNOWLEDGEBASE <span class="tag">Production v1.0</span></div>
      <div class="tech-stack">Vue 3 Computed + Async Stream + Vector Simulation</div>
    </header>

    <div class="rag-container">
      
      <section class="control-panel">
        <div class="card mb-20">
          <h3>📦 向量数据库仿真录入 (Vector DB Simulator)</h3>
          <div class="input-group">
            <input v-model="newKeywords" placeholder="匹配关键字（半角逗号隔开，如：pinia,状态）" class="rag-input" />
            <textarea v-model="newContent" placeholder="输入本地私有知识切片（Chunk）内容..." class="rag-input min-textarea"></textarea>
            <button @click="addCustomChunk" class="btn btn-secondary">⚡ 向量化并注入本地库</button>
          </div>
        </div>

        <div class="card flex-1 flex-col">
          <h3>🔍 语义检索与切片召回 (Retrieval Engine)</h3>
          <div class="search-box">
            <label>1. 输入用户提问 (User Query):</label>
            <input 
              v-model="userQuery" 
              placeholder="请输入您的问题，例如：组件通信 / 状态管理 / 跨组件..." 
              class="rag-input highlight"
            />
          </div>

          <div class="chunks-section flex-1 flex-col">
            <p class="section-title">
              📊 向量空间检索结果: 
              <span class="count-badge" :class="{ 'badge-green': matchedChunks.length > 0 }">
                {{ matchedChunks.length }} 个 Chunk 召回
              </span>
            </p>
            
            <div class="chunks-list flex-1">
              <div v-if="matchedChunks.length === 0" class="empty-tip">
                ⚠️ 未检索到高相关性本地私有知识，大模型回复可能发生幻觉或触发拒绝机制。
              </div>
              <div 
                v-else
                v-for="(chunk, index) in matchedChunks" 
                :key="index" 
                class="chunk-item"
              >
                <div class="chunk-meta">
                  <span>📄 召回切片 #{{ index + 1 }}</span>
                  <span class="score">相似度得分 (Cosine): <strong>{{ chunk.score }}</strong></span>
                </div>
                <p class="chunk-content">{{ chunk.content }}</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section class="display-panel flex-col">
        <div class="card flex-1 flex-col mb-20">
          <h3>📝 实时组装系统 Prompt 上下文 (System Context Builder)</h3>
          <div class="prompt-box flex-1">
            <pre><code>{{ assembledPrompt }}</code></pre>
          </div>
        </div>

        <div class="card flex-1 flex-col">
          <div class="panel-header-action">
            <h3>🤖 RAG 检索增强大模型生成 (LLM Generation)</h3>
            <button 
              @click="streamLLMResponse" 
              :disabled="isGenerating || !userQuery.trim()" 
              class="btn btn-primary"
            >
              {{ isGenerating ? '⚡ 正在召回并流式解析...' : '🚀 将增强 Prompt 送入大模型' }}
            </button>
          </div>

          <div class="response-display flex-1" id="responseDisplay">
            <div v-if="!llmOutput && !isGenerating" class="empty-response-tip">
              点击上方按钮，体验在弱网环境下，前端如何控制 RAG 的“高频吐字与 Markdown 解析渲染节奏”...
            </div>
            <div v-else class="markdown-body" v-html="parsedMarkdown"></div>
          </div>
        </div>
      </section>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

// 1. 初始化仿真本地向量知识库
const vectorDatabase = ref([
  { keywords: ['传参', 'props', 'emit', '组件通信'], content: 'Vue 3 中父传子使用 defineProps 接收属性，子传父使用 defineEmits 派发自定义事件进行通信。', score: '0.9642' },
  { keywords: ['provide', 'inject', 'pinia', '跨组件', '状态管理'], content: '对于深层跨组件全局通信，推荐引入轻量级状态管理库 Pinia，或者使用原生的 Provide / Inject 依赖注入机制。', score: '0.8915' },
  { keywords: ['v-model', '双向绑定'], content: 'Vue 3 的 v-model 语法糖默认会被编译器编译为名为 modelValue 的 prop 属性和名为 update:modelValue 的自定义事件。', score: '0.7841' }
])

// 2. 状态变量
const userQuery = ref('组件通信')
const newKeywords = ref('')
const newContent = ref('')
const llmOutput = ref('')
const isGenerating = ref(false)

// 3. 动态录入私有知识切片
const addCustomChunk = () => {
  if (!newKeywords.value.trim() || !newContent.value.trim()) {
    alert('请填写完整的匹配关键字与知识切片内容！')
    return
  }
  const keys = newKeywords.value.split(/[,，]/).map(k => k.trim()).filter(Boolean)
  vectorDatabase.value.unshift({
    keywords: keys,
    content: newContent.value,
    score: (Math.random() * 0.1 + 0.88).toFixed(4)
  })
  newKeywords.value = ''
  newContent.value = ''
}

// 4. 核心 RAG 检索
const matchedChunks = computed(() => {
  if (!userQuery.value.trim()) return []
  return vectorDatabase.value.filter(chunk => 
    chunk.keywords.some(keyword => userQuery.value.toLowerCase().includes(keyword.toLowerCase()))
  )
})

// 5. Prompt 编排器
const assembledPrompt = computed(() => {
  const systemRole = `【System Role说明】\n你是一个极为严谨的私有知识库助手。请严格基于以下给定的【已知参考资料】回答用户问题。如果已知资料中没有提及相关信息，请委婉拒绝回答，绝不可胡乱编造任何未知细节。\n`
  const divider = `\n------------------------------------------------------------\n`
  
  let context = `【已知参考资料 (RAG Context Chunks)】:\n`
  if (matchedChunks.value.length > 0) {
    matchedChunks.value.forEach((chunk, i) => {
      context += `[资料分块 ${i + 1}] (Cosine Score: ${chunk.score})\n内容: ${chunk.content}\n\n`
    })
  } else {
    context += `（⚠️ 警告：当前未匹配到任何私有知识切片，大模型被限制回答）\n`
  }

  const querySection = `【用户真实提问 (User Query)】:\n${userQuery.value}\n`
  const tail = `【回答输出规范】:\n请基于上述参考资料进行解答，并使用标准的 Markdown 语法（如：使用 **粗体** 突出重点，使用 \`代码块\` 标注技术API）进行排版输出。`

  return `${systemRole}${divider}${context}${divider}${querySection}${divider}${tail}`
})

// 6. 仿真大模型 SSE 异步流式吐字
const rawMarkdownTemplate = `### 🤖 知识库助手已就绪

根据为您检索并匹配到的 **3 个本地私有知识切片**，为您做出以下严谨解答：

1. **核心API应用**：
   在 Vue 3 组合式 API 中，组件间基础通信推荐使用 \`defineProps\` 和 \`defineEmits\`。这属于标准的单向数据流契约。
   
2. **进阶架构建议**：
   若涉及跨层级组件通信，为避免 “Prop 逐层透传” 导致的响应式追踪混乱，建议立刻升级至 \`Provide / Inject\` 架构或使用 **Pinia** 统一管理状态。

*以上内容完全基于私有知识库检索生成，未触发公网幻觉。*`

const streamLLMResponse = () => {
  if (isGenerating.value) return
  isGenerating.value = true
  llmOutput.value = ''
  
  let currentText = ''
  let index = 0
  
  const timer = setInterval(async () => {
    if (index < rawMarkdownTemplate.length) {
      currentText += rawMarkdownTemplate[index]
      llmOutput.value = currentText
      index++
      
      await nextTick()
      const display = document.getElementById('responseDisplay')
      if (display) display.scrollTop = display.scrollHeight
    } else {
      clearInterval(timer)
      isGenerating.value = false
    }
  }, 35)
}

// 7. 原生正则 Markdown 解析器
const parsedMarkdown = computed(() => {
  let html = llmOutput.value
  if (!html) return ''
  
  html = html.replace(/^### (.*$)/gim, '<h3 class="md-h3">$1</h3>')
  html = html.replace(/\*\*(.*?)\*\*/g, '<strong class="md-bold">$1</strong>')
  html = html.replace(/`(.*?)`/g, '<code class="md-code">$1</code>')
  html = html.replace(/^\s*\d\.\s(.*$)/gim, '<li class="md-li">$1</li>')
  html = html.replace(/^\s*\*\s(.*$)/gim, '<li class="md-bullet">$1</li>')
  
  html = html.replace(/\n/g, '<br>')
  return html
})
</script>

<style scoped>
/* 暗黑及硬核科技感全屏骨架 */
.rag-wrapper {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  background-color: #0b0f19;
  color: #e2e8f0;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, sans-serif;
  overflow: hidden;
}
.rag-header {
  background: #111827;
  padding: 14px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #1e293b;
}
.logo { font-weight: bold; font-size: 16px; color: #10b981; letter-spacing: 1px; }
.tag { font-size: 10px; background: #065f46; padding: 2px 6px; border-radius: 4px; color: #34d399; margin-left: 6px; }
.tech-stack { font-size: 12px; color: #64748b; }

/* 容器布局 */
.rag-container { flex: 1; display: flex; padding: 16px; gap: 16px; height: calc(100vh - 55px); overflow: hidden; }
.control-panel { width: 420px; display: flex; flex-direction: column; height: 100%; }
.display-panel { flex: 1; display: flex; flex-direction: column; height: 100%; }

/* 卡片基底 */
.card { background: #111827; border-radius: 10px; border: 1px solid #1e293b; padding: 16px; box-sizing: border-box; overflow: hidden; }
.flex-col { display: flex; flex-direction: column; }
.flex-1 { flex: 1; }
.mb-20 { margin-bottom: 16px; }

.card h3 { margin: 0 0 12px 0; font-size: 14px; color: #38bdf8; letter-spacing: 0.5px; border-bottom: 1px solid #1e293b; padding-bottom: 6px; }

/* 表单组件 */
.input-group { display: flex; flex-direction: column; gap: 8px; }
.rag-input {
  background: #0b0f19;
  border: 1px solid #1e293b;
  border-radius: 6px;
  padding: 8px 12px;
  color: #fff;
  font-size: 13px;
  outline: none;
  font-family: inherit;
}
.rag-input:focus, .rag-input.highlight { border-color: #10b981; }
.min-textarea { height: 50px; resize: none; }

/* 按钮设计 */
.btn { border: none; padding: 10px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; transition: all 0.2s; }
.btn-primary { background: #10b981; color: #0b0f19; width: auto; padding: 8px 16px; }
.btn-primary:hover { background: #34d399; }
.btn-primary:disabled { background: #334155; color: #64748b; cursor: not-allowed; }
.btn-secondary { background: #1e293b; color: #38bdf8; border: 1px solid #334155; }
.btn-secondary:hover { background: #334155; }

/* 检索视图 */
.search-box label { font-size: 12px; color: #94a3b8; display: block; margin-bottom: 6px; }
.chunks-section { margin-top: 14px; overflow: hidden; }
.section-title { font-size: 12px; color: #94a3b8; margin: 0 0 8px 0; display: flex; justify-content: space-between; }
.count-badge { background: #1e293b; color: #94a3b8; padding: 1px 6px; border-radius: 4px; font-size: 11px; }
.count-badge.badge-green { background: #065f46; color: #34d399; }

.chunks-list { background: #0b0f19; border-radius: 6px; border: 1px solid #1e293b; padding: 10px; overflow-y: auto; }
.empty-tip { color: #f59e0b; font-size: 11px; line-height: 1.5; text-align: center; padding: 20px 10px; }
.chunk-item { background: #111827; border: 1px solid #1e293b; border-left: 3px solid #10b981; padding: 10px; border-radius: 4px; margin-bottom: 10px; }
.chunk-meta { display: flex; justify-content: space-between; font-size: 11px; color: #64748b; margin-bottom: 4px; }
.score strong { color: #34d399; }
.chunk-content { margin: 0; font-size: 12px; color: #cbd5e1; line-height: 1.5; }

/* 右侧组件样式 */
.prompt-box { background: #0b0f19; border-radius: 6px; border: 1px solid #1e293b; padding: 10px; overflow-y: auto; }
.prompt-box pre { margin: 0; white-space: pre-wrap; font-size: 12px; color: #94a3b8; line-height: 1.4; }

.panel-header-action { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #1e293b; padding-bottom: 6px; margin-bottom: 12px; }
.panel-header-action h3 { border: none; margin: 0; }

.response-display { background: #0b0f19; border-radius: 6px; border: 1px solid #1e293b; padding: 16px; overflow-y: auto; }
.empty-response-tip { color: #475569; font-size: 12px; text-align: center; padding-top: 60px; line-height: 1.6; }

/* 样式注入 */
:deep(.md-h3) { color: #38bdf8; margin: 0 0 12px 0; font-size: 15px; border-bottom: 1px solid #1e293b; padding-bottom: 4px; }
:deep(.md-bold) { color: #10b981; font-weight: bold; }
:deep(.md-code) { background: #1e293b; color: #f43f5e; padding: 2px 5px; border-radius: 4px; font-size: 11px; margin: 0 2px; }
:deep(.md-li) { color: #cbd5e1; font-size: 13px; line-height: 1.6; margin-bottom: 6px; list-style-type: decimal; margin-left: 14px; }
:deep(.md-bullet) { color: #cbd5e1; font-size: 13px; line-height: 1.6; margin-bottom: 4px; list-style-type: disc; margin-left: 14px; }
</style>