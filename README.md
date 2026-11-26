# 心理健康助手 · 前端

Vue 3 + Vite + Element Plus 实现的用户端与管理端界面。

## 技术栈

- Vue 3（`<script setup>` SFC）
- Vite 7
- Element Plus 2、ECharts 6
- Pinia、Vue Router 4
- `@microsoft/fetch-event-source`（流式对话，POST + token 头）
- WangEditor（富文本编辑，管理端文章）

## 页面

- 用户端：首页、咨询对话（流式）、情绪日志（提交 + 我的记录分页）、情绪花园、知识文章
- 管理端：登录、咨询会话分页、情绪日志管理、知识库管理、数据看板

## 运行

```bash
npm install
npm run dev
```

默认 `vite.config.js` 中 `/api` 代理到 `http://localhost:1236`（本地后端）。如需指向其他环境，改 `server.proxy` 的 `target`。

## 与后端的接口约定

- 统一前缀 `/api`，请求头携带 `token`
- 响应结构 `{ code, msg, data }`，`code === '200'` 视为成功，`code === '-1'` 视为登录失效
- 流式接口 `POST /api/psychological-chat/stream`，`event: message` 推内容、`event: done` 结束

## 构建

```bash
npm run build
```
