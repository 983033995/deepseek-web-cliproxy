# deepseek-web-cliproxy

将 DeepSeek 网页版会话适配为 CLIProxyAPI 的独立 Provider Plugin，通过 OpenAI 兼容接口调用网页端模型。

## 项目定位

本项目面向个人、本机和实验性使用：复用用户已经登录的 DeepSeek Web 会话，不需要 DeepSeek API Key。它不是官方 API，也不是面向多人共享或生产流量的免费 API 网关。

核心目标是把网页端能力接入 CLIProxyAPI：

```text
OpenAI Client → CLIProxyAPI → deepseek-web plugin → chat.deepseek.com Web API
```

首版只承诺文本 Chat、流式 SSE、Reasoner 映射、模型列表和可诊断错误；Responses API、文件、视觉、搜索和工具调用属于后续范围。

## 重要边界

- 上游使用 DeepSeek 私有网页接口，接口、PoW、风控和会话行为可能随时变化。
- “免费”仅表示不产生 DeepSeek API 账单，仍受网页账号额度、并发和服务条款约束。
- 不保存明文密码；默认使用用户提供的 User Token 或浏览器会话导入。
- 默认单账号、低并发、本机监听；生产部署、公共网关和绕过风控不在范围内。

## 文档

- [架构设计](docs/architecture.md)
- [需求范围](docs/requirements.md)
- [里程碑](docs/roadmap.md)
- [风险与应对](docs/risks.md)
- [本地 Codex 实施指南](docs/local-codex-guide.md)

## 参考实现

需求分析参考 `booleamu/deepseek-mcp-server` 的 Web Client、PoW、Session 和 SSE 处理思路；CLIProxyAPI 侧应参考其当前 Plugin ABI 与 `workbuddy-cliproxy` 等同类 Provider Plugin。实现前必须以目标版本源码和官方文档重新核对接口，不把本 README 当作 ABI 规范。

## 验收定义（原型阶段）

文档完成即表示原型完成。代码阶段的最小验收是：本地 CLIProxyAPI 加载插件；`GET /v1/models` 返回声明的 DeepSeek Web 模型；`POST /v1/chat/completions` 能完成非流式和流式文本请求；失效会话、PoW 失败、上游限流和 SSE 异常均返回可诊断的 OpenAI 错误结构。
