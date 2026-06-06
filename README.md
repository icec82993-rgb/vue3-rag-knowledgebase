# Vue3 RAG Knowledge Base Demo

基于 Vue 3 + Vite 构建的智能知识库检索与 Prompt 调试平台，通过知识片段匹配模拟 RAG（Retrieval-Augmented Generation）流程，实现知识检索、Prompt 动态编排、大模型流式生成与结果展示。

---

## 📖 项目简介

本项目旨在可视化展示 RAG（检索增强生成）的核心工作流程，帮助理解知识库检索、上下文构建以及大模型生成回答之间的协作关系。

用户输入问题后，系统会从本地知识库中匹配相关知识片段，动态构建 Prompt 上下文，并调用大模型 API 生成回答，实现从检索到生成的完整链路展示。

---

## 🚀 技术栈

- Vue 3
- Composition API
- Vite
- Fetch API
- ReadableStream
- SSE（Server-Sent Events）
- Git

---

## ✨ 核心功能

### 1. 知识库管理

支持本地知识片段录入与维护。

```text
用户录入知识
    ↓
存入知识库
    ↓
等待匹配检索
```

### 2. 知识片段匹配

根据用户输入内容进行关键词匹配，筛选相关知识片段。

```text
用户问题
    ↓
关键词匹配
    ↓
召回相关知识片段
```

### 3. Prompt 动态编排

基于匹配到的知识片段自动构建 Prompt。

```text
System Prompt
      +
知识片段
      +
用户问题
      ↓
生成完整 Prompt
```

### 4. 大模型接口调用

基于 Fetch API 调用大模型接口。

```javascript
const response = await fetch(apiUrl)
```

### 5. 流式响应展示

利用 ReadableStream 实现实时内容解析与增量渲染。

```text
模型输出
    ↓
SSE数据流
    ↓
ReadableStream解析
    ↓
实时展示
```

### 6. Prompt 调试看板

支持查看：

- 用户输入
- 匹配知识片段
- Prompt 构建结果
- 模型生成结果

方便分析检索与生成过程。

---

## 📂 项目结构

```text
src
│
├── App.vue
│
├── 用户输入模块
├── 知识库匹配模块
├── Prompt构建模块
├── 模型调用模块
└── 流式输出模块
```

---

## 🎯 项目亮点

### Vue3 响应式状态管理

使用：

- ref
- computed
- nextTick

实现响应式数据流管理。

### Prompt 动态生成

根据用户问题自动拼接知识片段与上下文信息，完成 Prompt 编排。

### 流式文本渲染

基于：

- Fetch API
- ReadableStream
- TextDecoder

实现大模型实时输出效果。

---

## 🔄 RAG 流程示意

```text
用户提问
    │
    ▼
知识片段匹配
    │
    ▼
Prompt动态构建
    │
    ▼
大模型API调用
    │
    ▼
流式响应解析
    │
    ▼
结果展示
```

---

## 💡 学习收获

通过本项目掌握：

- Vue3 Composition API 开发模式
- 响应式状态管理
- computed 派生状态设计
- Fetch API 网络请求
- ReadableStream 流式响应处理
- Prompt 工程基础
- RAG 工作流程理解
- Git 版本管理与 Rebase 使用

---

## 🛠️ 本地运行

```bash
# 安装依赖
npm install

# 启动开发环境
npm run dev

# 项目构建
npm run build
```

---

## 📌 项目定位

本项目定位为基于 Vue3 的 RAG 流程演示与 Prompt 调试平台，重点展示知识检索、上下文构建、流式响应解析与生成结果展示的完整链路，适用于学习 Vue3、Prompt Engineering 与 RAG 基础流程。
