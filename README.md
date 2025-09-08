# vsse

Lightweight front-end SSE manager with single-connection multiplexing.

中文简介：统一 SSE 长连接、多任务共享；POST 发起任务并通过 requestId 路由 SSE 消息到对应回调；
支持全局与单次 POST 配置（headers/timeout/credentials/token）。

## 安装

```shell
- npm i vsse
```

## 快速开始
```js
import { SSEClient } from "vsse"; // 或相对路径 ./src/index.js

const sse = new SSEClient({
  url: "/sse?userId=alice",           // 必填：SSE 服务地址（可带查询参数）
  eventName: "notify",                // 可选：服务端自定义事件名；默认 'message'，若后端发 'notify'，在此指定

  // 空闲与心跳（常用）
  idleTimeout: 30_000,                // 可选：仅当“无任何监听器”时，空闲多久自动断开（ms）；设 0 可关闭空闲断开
  withHeartbeat: true,                // 可选：默认 true；启用心跳检测（任意消息或 event=ping 视为“活动”）
  expectedPingInterval: 15_000,       // 可选：默认 15_000ms；若 2×该值内未收到“任何消息/心跳”，将发起重连

  // POST 全局默认（连接本身不带自定义 headers）
  defaultHeaders: { "X-App-Version": "1.0.0" }, // 可选：POST 默认请求头（SSE 连接本身不支持自定义 headers）
  defaultTimeout: 10_000,             // 可选：POST 默认超时（ms），可被单次调用覆盖
  credentials: "include",            // 可选：POST 默认凭据（include/same-origin/omit）
  token: "your-jwt-token",           // 可选：POST 默认 Authorization: Bearer <token>
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

---

## 最小配置示例（行尾含中文注释）
```js
import { SSEClient } from 'vsse';

const sse = new SSEClient({
  url: '/sse?userId=alice',           // 必填：SSE 服务地址（可带查询参数）
  eventName: 'notify',                // 可选：后端自定义事件名；默认 'message'；若服务端发 'notify'，在此指定

  // 空闲与心跳（常用）
  idleTimeout: 30_000,                // 可选：仅当“无任何监听器”时空闲多久自动断开（ms）；设 0 可关闭空闲断开
  withHeartbeat: true,                // 可选：默认 true；启用心跳检测（任意消息或 event=ping 视为“活动”）
  expectedPingInterval: 15_000,       // 可选：默认 15_000ms；若 2×该值内未收到“任何消息/心跳”，将发起重连

  // POST 全局默认（连接本身不带自定义 headers）
  defaultHeaders: { 'X-App': 'demo' },// 可选：POST 默认请求头（SSE 连接不支持自定义 headers）
  defaultTimeout: 10_000,             // 可选：POST 默认超时（ms），可被单次调用覆盖
  credentials: 'include',             // 可选：POST 默认凭据（include/same-origin/omit）
  token: 'your-jwt-token',            // 可选：POST 默认 Authorization: Bearer <token>
});
```

## 完整配置清单（含默认值与行为说明）
```js
import { SSEClient } from 'vsse';

const sse = new SSEClient({
  // ========== 连接与事件 ========== 
  url: '/sse?userId=alice',            // 必填：SSE 服务地址
  eventName: 'message',                // 默认 'message'；若服务端使用 'notify'，改为 'notify'

  // ========== 空闲与心跳 ========== 
  idleTimeout: 30_000,                 // 默认 30_000ms；仅在“没有任何监听器”时按此关闭；设 0 关闭空闲断开
  withHeartbeat: true,                 // 默认 true；启用心跳监测（收到任意消息或 event=ping 视为心跳）
  expectedPingInterval: 15_000,        // 默认 15_000ms；超时判定为 2×该值内未收到任何消息⇒重连

  // ========== 凭据与安全 ========== 
  sseWithCredentials: false,           // 默认 false；SSE 连接是否携带 Cookie；跨域需后端返回 ACAC 且精确 ACAO

  // ========== POST 全局默认（单次可覆盖） ========== 
  defaultHeaders: {                    // 可选：POST 默认请求头
    'Content-Type': 'application/json',// 库会设置；可自定义再合并/覆盖
    'X-App': 'demo',
  },
  defaultTimeout: 10_000,              // 默认 10_000ms；POST 超时，可被单次 options.timeout 覆盖
  credentials: 'include',              // 默认 undefined；POST 凭据（include/same-origin/omit）
  token: undefined,                    // 默认 undefined；POST 默认 Authorization: Bearer <token>

  // ========== 连接保护与重连 ========== 
  maxListeners: 1000,                  // 默认 1000；监听器数量上限，防内存泄漏
  reconnectBackoff: {                  // 指数退避 + 抖动
    baseMs: 1000,                      // 起始基准延迟（ms）
    maxMs: 15_000,                     // 最大延迟（ms）
    factor: 1.8,                       // 指数因子
    jitter: 0.3,                       // 抖动比例 [0,1]，避免雪崩
  },
});
```

## 快速开始（含单次覆盖）
```js
// 发起 POST 并监听 SSE；第四个参数为“单次 options”，优先级高于全局默认
const controller = new AbortController();
const { requestId, unsubscribe } = await sse.postAndListen(
  '/api/doA',                          // POST URL
  { foo: 'bar' },                      // 请求体（库会附加 requestId）
  ({ event, payload }) => {            // 事件回调：progress/done/error/ping/自定义
    if (event === 'progress') {
      // 处理进度
    } else if (event === 'done') {
      // 处理完成（自动取消该 requestId 的监听）
    } else if (event === 'error') {
      // 处理错误（自动取消该 requestId 的监听）
    }
  },
  {
    headers: { 'X-Debug': '1' },       // 单次 POST 请求头，覆盖 defaultHeaders
    timeout: 5_000,                    // 单次 POST 超时（ms），覆盖 defaultTimeout
    credentials: 'include',            // 单次 POST 凭据，覆盖全局 credentials
    token: 'special-task-token',       // 单次 POST Authorization: Bearer <token>，覆盖全局 token
    signal: controller.signal,         // 外部取消控制；controller.abort() 可中断本次 POST
    requestId: crypto.randomUUID(),    // 可选：自定义请求 ID；不传则自动生成
  }
);

// 随时手动取消监听（仅影响前端回调路由；不影响其他任务）
// unsubscribe();

// 如需中断正在进行的 POST
// controller.abort();
```

## 选项与默认值总览（行为语义）
- url：SSE 服务地址（必填）。
- eventName：默认 "message"；后端若用 "notify"，需设为 "notify" 才能被 addEventListener 捕获。
- idleTimeout：默认 30_000ms；仅在“无任何监听器”时按此关闭；设 0 可关闭空闲断开。
- sseWithCredentials：默认 false；SSE 连接是否携带 Cookie。跨域需服务端返回：
  - Access-Control-Allow-Origin: https://your.app
  - Access-Control-Allow-Credentials: true
- defaultHeaders/defaultTimeout/credentials/token：仅作用于 POST。
- withHeartbeat：默认 true；启用心跳检测。
- expectedPingInterval：默认 15_000ms；超过 2×该值未收到“任何消息/心跳”即判定超时并重连。
- maxListeners：默认 1000；监听器上限。
- reconnectBackoff：默认 { baseMs: 1000, maxMs: 15_000, factor: 1.8, jitter: 0.3 }；控制断线后的重连延迟。

合并/优先级规则（POST）：
- headers：defaultHeaders < options.headers（单次覆盖全局）。
- token：this.opts.token < options.token（单次优先）。
- timeout：this.opts.defaultTimeout < options.timeout（单次优先）。
- credentials：this.opts.credentials < options.credentials（单次优先）。

## 心跳与重连（关键时序）
- 活动的定义：收到任意 SSE 消息或收到 event=ping。
- 超时阈值：2 × expectedPingInterval。超过此阈值未收到“任何消息”，触发重连。
- 检测粒度：内部每 5s 检查一次心跳超时，因此触发时刻可能存在最多 ~5s 的检测延迟。
- 重连退避：断开后按照 reconnectBackoff 进行指数退避并带抖动，避免雪崩；首次大约 baseMs。
- 注意：后台标签页可能被浏览器节流，表现为定时器/回调延迟。

服务端心跳示例（建议每 expectedPingInterval 发送一次）：
```
event: notify
data: {"event":"ping"}
```

## 生命周期管理
```js
// 动态更新配置（变更 url 会自动重连）
sse.updateConfig({ url: '/sse?userId=bob', expectedPingInterval: 20_000 });

// 手动重连（例如网络恢复后立即尝试）
sse.reconnect();

// 手动关闭（保留状态；未来有监听器时可再连）
sse.close();

// 完全销毁（组件卸载/页面离开时调用，移除全局事件监听与定时器）
sse.destroy();
```

## 服务端事件格式与路由约定
- 建议每条 SSE data 为 JSON：{ requestId, event, payload }。
- 若 data 无 event 字段，将回退使用原生 SSE 事件名 ev.type（如 message/notify）。
- event 为 'done' 或 'error' 时，该 requestId 的监听会自动移除。

示例：
```
event: notify
data: {"requestId":"<uuid>","event":"progress","payload":{"ratio":0.5}}

event: notify
data: {"requestId":"<uuid>","event":"done","payload":{"result":"ok"}}
```

## CORS、凭据与 EventSource 限制
- EventSource 连接不支持自定义请求头，也无法直接携带 Authorization。Authorization 仅对 POST 生效。
- 如需让连接层携带 Cookie，设置 sseWithCredentials=true，且服务端必须：
  - Access-Control-Allow-Origin: 精确来源（不能是 *）
  - Access-Control-Allow-Credentials: true
- 如必须在连接层做 Bearer 鉴权：
  - 方案 A：使用基于 Cookie 的会话；
  - 方案 B：改用 fetch + ReadableStream 自行解析 SSE（本库当前未内置该模式）。

## 常见问题（FAQ）
- 为什么我设置的 expectedPingInterval 和“实际断开/重连间隔”不一致？
  - 判定超时阈值是 2×expectedPingInterval；再加上定时检测步进（5s）与退避延迟，肉眼观测会更长。
- 为什么连接会自动断开？
  - 可能因为：心跳超时；网络 offline；服务端关闭；无监听器且达到 idleTimeout。
- 如何关闭“无监听器时”的空闲断开？
  - 将 idleTimeout 设为 0。
- 为什么看不到 SSE 连接上的 Authorization 头？
  - EventSource 限制所致。Authorization 仅用于 POST；连接层鉴权请用 Cookie 或自实现流式解析。
- 跨域携带 Cookie 不生效？
  - 检查服务端是否返回 Access-Control-Allow-Credentials: true 且 Access-Control-Allow-Origin 为精确源而非 *。

## 排查清单（出现“时断时续/延迟重连”时）
- 服务端是否按期发送心跳（或至少有业务消息）？
- eventName 是否与服务端一致（message vs notify）？
- 是否考虑了“2×expectedPingInterval + 5s 检查步进 + 退避延迟”的综合效果？
- 标签页是否在后台导致节流？
- 是否在大量任务场景触及 maxListeners 限制？

## 多任务复用与懒连接
- 仅当存在至少一个监听器时才建立 SSE 连接（懒连接）。
- 多个 postAndListen 共享同一连接，通过 requestId 路由到各自回调。

```js
const a = await sse.postAndListen('/api/a', {...}, onA);
const b = await sse.postAndListen('/api/b', {...}, onB);

// 取消其中一个
a.unsubscribe();
// 若全部取消且达到 idleTimeout，将自动关闭连接
```

## 提交到仓库的 PR 建议（仅文档增强）
- Commit（Conventional Commits）：
  - docs(vsse): expand README with heartbeat/backoff/credentials and usage examples
- CHANGELOG（vsse/CHANGELOG.md → [Unreleased]）：
  - Added — Expanded README: heartbeat, backoff, sseWithCredentials, per-call overrides, lifecycle, FAQ
- 版本：不涉及行为变更，可作为文档更新并入后续发版（无须立即改版本号）。
