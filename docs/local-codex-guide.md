# 本地 Codex 实施指南

## 开始前

1. 在目标机器准备 CLIProxyAPI 源码或已发布版本，确认具体 Plugin ABI、配置键和加载目录。
2. 阅读同版本 Provider Plugin 示例，记录接口签名、构建标签、平台产物和错误协议。
3. 仅使用测试账号或明确授权的个人网页账号；不要把 token、cookie、密码写入仓库。
4. 建立协议记录：认证请求、session 创建、PoW challenge、completion SSE 的脱敏样本和时间戳。

## 推荐实现顺序

按 M1 → M2 → M3 → M4 进行。每个里程碑先写契约/适配测试，再接下一层；不要同时实现文件、工具调用或多账号。

目录职责建议：

```text
plugin/       CLIProxyAPI ABI 适配
provider/     OpenAI 请求/响应与模型注册
web/          DeepSeek Web transport、SSE、PoW
auth/         auth record、导入、脱敏和存储
session/      session 生命周期与并发隔离
diagnostics/  错误分类、日志和健康检查
```

## 验证顺序

先运行仓库实际提供的格式化、单元测试和构建命令；再把插件装入本地 CLIProxyAPI，依次验证 `/v1/models`、非流式 Chat、流式 Chat、取消、token 失效和 PoW 失败。命令以目标仓库文档为准，不预设包管理器或 Make 目标。

## 完成标准

- 插件可加载且默认关闭或显式启用。
- 两种 Chat 模式均能通过真实 CLIProxyAPI 端点验收。
- 错误、日志、凭据和并发行为符合本文档。
- README 的能力声明与真实测试结果一致。
- 未支持能力明确返回错误，而不是静默降级。
