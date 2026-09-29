# 架构设计

## 组件

```text
CLIProxyAPI
  ├─ ModelProvider / AuthProvider / Executor
  └─ deepseek-web plugin
       ├─ OpenAI Request Translator
       ├─ Web Session Manager
       ├─ PoW Solver Adapter
       ├─ DeepSeek Web Transport
       ├─ SSE Decoder
       └─ OpenAI Response Mapper
```

插件保持独立仓库和独立发布物（macOS `.dylib`、Linux `.so`、Windows `.dll`，具体扩展名以 CLIProxyAPI ABI 为准）。不要 fork CLIProxyAPI，也不要修改其内部 runtime。

## 请求生命周期

1. Executor 校验模型、消息和流式参数。
2. Translator 将 OpenAI `messages[]` 转成网页 prompt；首版保留角色边界并明确 system/developer 的降级规则。
3. Session Manager 首版每个请求创建独立 `chat_session_id`，避免并发串话；后续再用历史指纹安全复用。
4. PoW Solver 获取 challenge 并生成 `x-ds-pow-response`。PoW 算法必须可替换，优先 WASM/官方 worker，禁止把易变实现散落在 Executor。
5. Transport 调用 `chat_session/create`、completion 等网页接口，处理认证、超时、重试和取消。
6. SSE Decoder 将 thinking、answer、usage、终止事件转换为 OpenAI chunk；非流式请求复用同一事件聚合器。
7. Mapper 输出 `chat.completion` 或 `chat.completion.chunk`，reasoning 内容放入兼容字段并保留原始诊断信息。

## 认证与状态

认证优先级：显式 User Token/浏览器会话导入 > 已保存 auth record；不在插件中实现邮箱密码自动登录。Auth record 至少包含 token、必要 cookies、账号标识、过期时间和 schema version，并使用系统密钥链或 CLIProxyAPI 的安全存储。

首版状态隔离：账号、模型、请求 ID、Web session ID 分开记录；任何日志都必须脱敏。并发限制默认 1，队列和取消由 Executor 负责。

## 兼容策略

- 模型 ID 使用稳定的插件前缀，如 `deepseek-web-chat`、`deepseek-web-reasoner`；真实网页模型名放在内部映射表。
- 不伪造原生 Function Calling。收到 `tools` 时首版明确返回 unsupported；后续只能通过可审计的 prompt 协议实现，并标注非严格兼容。
- 文件、视觉、搜索、Responses API 与 structured output 均以能力探测和独立里程碑加入。
