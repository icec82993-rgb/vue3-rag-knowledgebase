# Vue 3 Secure RAG Pipeline & SSE Stream Hub
> **基于 Vue 3 (Composition API) 与原生的 RAG 增强提示词编排及真·大模型 SSE 流式响应看板**

## 🌟 核心工程背景 (Engineering Context)
在企业级生产环境中，大模型（LLM）应用的落地普遍面临三大工程挑战：
1. **RAG 链路透明度低**：用户输入提问后，向量切片如何召回、系统 Prompt 如何动态拼接，在传统架构中属于“黑盒”，给提示词调优（Prompt Engineering）带来极大困难。
2. **长文本响应卡顿**：大模型单次生成可能耗时数十秒，若采用传统的 HTTP 轮询或单次等待返回，会导致前端界面长时间陷入白屏死锁，极度损害用户体验。
3. **第三方 SDK 臃肿**：引入庞大的重量级大模型服务链 SDK 会导致前端打包体积膨胀。

本项目完全使用 **Vue 3 响应式机制** 底层重构，从零实现了**仿真向量知识注入、动态检索召回、实时 Prompt 组装**的全链路，并采用纯原生 Web API 攻克了 **DeepSeek 大模型 Server-Sent Events (SSE) 异步流式文本流控制**，是一款专为 AI 研发端量身打造的 RAG 调试控制台。

---

## 🛠️ 硬核技术栈与架构设计 (Architecture & Tech Stack)
* **前端骨架**：Vue 3 (SFC) + Vite + Tailwind/Custom Dark Sci-Fi CSS（全屏硬核科技暗黑视觉）
* **数据驱动层**：基于 Vue 3 的 `ref` 与 `computed` 构建高频响应式状态机。
* **数据流通信**：标准 Web Fetch API + `ReadableStream`（无第三方 SDK 依赖）
* **Markdown 解析层**：高性能原生正则表达式动态文本流清洗器。

---

## 🔥 核心技术亮点与面试高光点 (Key Features & Interview Highlights)

### 1. 基于 `ReadableStream` 的底层 SSE 真·流式吐字控制
* **工程痛点**：大模型流式接口返回的是 `text/event-stream` 类型数据。
* **解法与亮点**：告别市面上常见的 `setInterval` 假模拟。本项目直接通过原生 `fetch` 建立流式 POST 管道，利用 `response.body.getReader()` 逐块读取底层二进制流。通过 `TextDecoder` 动态转换并处理粘包、断包碎片，提取 `data: ` 前缀背后的 Delta 文本内容，实现低延迟、零内存积压的秒级响应渲染。

### 2. 响应式自适应 RAG 语义检索与切片召回引擎
* **设计细节**：利用 Vue 3 的 `computed` 依赖收集特性，对模拟向量库（Vector DB Table）实现秒级多关键词模糊检索沙箱。当用户输入 `User Query` 时，检索引擎实现微秒级切片自适应计算与排序，动态输出 Cosine 相似度得分，直观展示召回机制。

### 3. 动态系统 Prompt 上下文自动组装机制
* **设计细节**：严格遵循大