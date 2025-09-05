# 项目状态（STATUS / ROADMAP）

## 能力矩阵（当前）
- [x] 聊天对话（OpenAI，非流式）
- [x] 流式对话（Replicate + SSE）
- [x] 右侧需求卡渲染（SSE 推送 HTML）
- [x] 航班 JSON → HTML 渲染
- [ ] 单元测试与契约测试
- [ ] CI（Windows + Ubuntu）
- [ ] 账号体系与会话持久化

## 路线图（计划）
- 近期（0.1.x）
    - 移除硬编码密钥，统一使用环境变量
    - 增加 API 合同测试与 SSE 行为测试
    - 文档与类型声明完善
- 中期（0.2.x）
    - 引入缓存与并发去重（inflight 合并）
    - 性能与慢查询日志（命中率、p95 指标）
- 远期（1.x）
    - 稳定公开 API，发布初版
    - 提供插件化模型供应商适配

注：路线图更新需同步 CHANGELOG 的 [Unreleased]。