# 需求范围

## MVP 必须有

| 编号 | 需求 | 验收信号 |
|---|---|---|
| F-01 | 以 CLIProxyAPI Plugin ABI 注册 Provider、Auth、Executor | 启动日志显示插件已加载 |
| F-02 | User Token/浏览器会话导入，不接收明文密码 | 能保存、校验、吊销 auth record |
| F-03 | `GET /v1/models` 映射网页模型 | 返回稳定 ID、能力和来源 |
| F-04 | OpenAI Chat Completions 非流式 | 文本响应、finish reason、错误结构正确 |
| F-05 | OpenAI Chat Completions 流式 | SSE chunk 顺序正确，可被取消 |
| F-06 | thinking/reasoner 映射 | 思考内容与答案不混淆，或按客户端能力降级 |
| F-07 | PoW、超时、限流、token 失效处理 | 错误可分类、可重试性明确 |
| F-08 | 脱敏日志与 request ID | 日志可定位请求且无 token/cookie/prompt 泄露 |

## MVP 明确不做

邮箱密码自动登录、多账号轮换、公共部署、Function Calling、Responses API、文件上传、视觉、网页搜索、结构化输出保证、自动绕过 WAF 或频率限制。

## 非功能需求

- 单请求默认超时可配置；取消必须停止上游读取。
- 同一账号默认串行，避免网页会话竞态。
- 失败不静默吞掉，返回稳定错误码、上游状态和 request ID。
- 认证信息不进入 Git、core dump、普通日志或错误消息。
- 适配层与网页传输层分离，便于网页协议变化时替换。
