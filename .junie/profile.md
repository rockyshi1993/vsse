# Junie Profile（vsse）

## 关键目录与入口
- 源码入口：src\index.js（导出 SSEClient）
- 实现文件：src\sse-client.js（单连接复用、requestId 路由、POST 配置覆盖、idle/心跳/重连）
- 包清单：package.json（"type": "module"，无构建，files: ["src"]）

## 运行与测试（Windows/PowerShell）
- 环境：Node 18.x/20.x（LTS）
- 安装依赖：本包为纯 JS，无外部依赖；在仓库根或 vsse 目录执行 `npm ci`（如需要）
- 测试：`npm test`（当前为占位，执行 `node --version`）。建议后续补充浏览器端伪测试或 E2E（例如使用 Playwright）。

## 使用与开发
- 本地开发：直接编辑 src\sse-client.js、src\index.js；无需构建。
- 在其他前端项目中引用：
  - 方式1：相对路径导入 `import { SSEClient } from '../vsse/src/index.js'`
  - 方式2：发布到 npm 后 `import { SSEClient } from 'vsse'`
- 重要说明：浏览器原生 EventSource 不支持自定义 headers；本库仅对 POST 支持 headers/token。若需在 SSE 连接携带 Authorization，可改为 fetch+ReadableStream 自行解析（未来可选提供）。

## 构建与发布
- 构建：无构建流程（`npm run build` -> echo 占位）。
- 发布包体：package.json `files` 仅包含 `src`，入口 `main/module` 指向 `src/index.js`。
- 版本与日志：遵循 SemVer；对外可见变更需同步更新 CHANGELOG.md 的 [Unreleased] 或新增版本块。

## 测试建议（待补充）
- 连接管理：首次 postAndListen 触发连接；idle 到期断开；用户交互触发重连。
- 路由分发：并发多个 requestId，事件不串扰；done/error 自动清理。
- POST 行为：headers/timeout/credentials/token 单次覆盖优先级正确；超时触发 Abort。
- 心跳与重连：心跳超时触发重连；SSE error 指数退避。

## 安全与配置
- 不记录 token/凭据到日志与注释；避免在 URL 中长期暴露签名。
- 如需凭据，优先使用 Cookie + withCredentials；CORS 需设置具体 Allow-Origin，允许凭据。

## 文档联动
- README：公共 API/默认值/示例变更时更新；
- CHANGELOG：每次对外可见变更都更新（Added/Changed/Fixed/Deprecated/Removed 等）；
- 若计划增强（如 fetch 解析 SSE、跨标签页共享等），在 STATUS/ROADMAP（如在根或项目内维护）同步。

## 例外与覆盖（相对通用规范）
- 例外：本包为纯浏览器端 JS 库，无构建与类型声明；测试暂为占位脚本。
- 影响面说明：不影响使用者 API；后续补充测试与包体检查（可在 CI 加上 `npm pack`）后，更新本 Profile。
