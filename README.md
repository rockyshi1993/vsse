# vsse

Lightweight front-end SSE manager with single-connection multiplexing.

中文简介：统一 SSE 长连接、多任务共享；POST 发起任务并通过 requestId 路由 SSE 消息到对应回调；
支持全局与单次 POST 配置（headers/timeout/credentials/token）。

## 安装
- 本地使用：本包为纯 JS，无需构建。直接以相对路径或包名引入即可。
- 作为库发布到 npm 后：`npm i vsse`

## 快速开始
```js
import { SSEClient } from "vsse"; // 或相对路径 ./src/index.js

const sse = new SSEClient({
  url: "/sse?userId=alice",
  eventName: "notify",           // 若你的服务端事件名为 notify
  idleTimeout: 30_000,
  defaultHeaders: { "X-App-Version": "1.0.0" },
  defaultTimeout: 10_000,
  credentials: "include",
  token: "your-jwt-token",
});

// 发起 POST 并监听 SSE（第四个参数为单次配置，可覆盖默认）
const { requestId, unsubscribe } = await sse.postAndListen(
  "/api/doA",
  { foo: "bar" },
  ({ event, payload }) => {
    // 你的后端 payload 示例：{ type: 'need' | 'text', content: ... }
    if (payload && payload.type === "need") {
      // 渲染需要型内容
    } else if (payload && payload.type === "text") {
      // 流式文本
    }
  },
  {
    timeout: 5000,
    headers: { "X-Debug": "true" },
    token: "special-task-token",
  }
);

// 手动取消前端监听（可选）
unsubscribe();
```

## API 摘要
- new SSEClient(options)
  - url: SSE 服务端地址；
  - eventName: 服务端事件名（默认为 "message"，你的后端用 "notify" 则设置为 notify）；
  - idleTimeout: 空闲多久断开；
  - sseWithCredentials: 是否在 SSE 连接中携带凭据（Cookie）；默认 false；仅当同域会话或服务端已允许凭据时再开启；
  - defaultHeaders/defaultTimeout/credentials/token: POST 的全局默认。
- postAndListen(postUrl, body, onEvent, options?)
  - options.headers/timeout/credentials/token/signal/requestId 支持按次覆盖；
  - 返回 { requestId, unsubscribe }。
- updateConfig(patch)
- close()/reconnect()
- destroy(): 释放全局事件监听与定时器，组件卸载或不再使用时调用（防止内存泄漏）。

## 注意
- 浏览器原生 EventSource 不支持自定义 headers；本库仅对 POST 支持 headers/token。
- SSE 连接的 withCredentials 现在可配：默认 false；仅当后端开启 `Access-Control-Allow-Credentials: true` 且需要携带 Cookie 时，才将 `sseWithCredentials` 设为 true。
- 发起 POST 若失败（网络/CORS/非 2xx），本库会自动清理对应监听器并抛错，请在调用处捕获并提示用户。
- 如需让 SSE 连接也携带 Authorization，可改为 fetch+ReadableStream 自行解析 SSE（后续版本可选提供）。
