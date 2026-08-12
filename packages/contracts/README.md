# Contracts

这里存放前后端共享的公开契约，例如：

- OpenAPI 生成的 TypeScript 客户端；
- Agent、Tool、Run、Trace、Evaluation 等领域 DTO；
- SSE/WebSocket 事件类型；
- JSON Schema 与版本兼容说明。

初期以 FastAPI OpenAPI 为单一事实来源，后续通过代码生成同步到 Web，避免手写两套类型。

