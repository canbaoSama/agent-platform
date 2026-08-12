# Agent Platform

面向企业场景的 AI Agent 平台基础工程，目标是把模型、工具、知识、记忆、工作流、评测与治理能力组合成可观测、可控制、可扩展的业务系统。

## 快速开始

```powershell
Copy-Item .env.example .env
docker compose -f infra/docker-compose.yml up -d
pnpm install
pnpm dev:web
```

另开一个终端启动 API：

```powershell
cd apps/api
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
uvicorn app.main:app --reload --port 8000
```

- Web: http://localhost:5173
- API 文档: http://localhost:8000/docs
- API 健康检查: http://localhost:8000/api/v1/health

## 文档

- [平台概括](docs/平台概括.md)
- [完整技术说明](docs/完整技术说明.md)
- [AI Agent 与 LLM 岗位技术学习及面试手册](docs/AI-Agent与LLM岗位技术学习及面试手册.md)

当前版本是架构骨架：包含可运行的 Vue 入口、FastAPI 健康检查、共享契约目录和本地依赖编排，尚未接入真实模型与 Agent 工作流。
