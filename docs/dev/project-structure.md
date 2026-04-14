# 工程结构与规范

## 目录结构（示例）

```text
repo/
├── cmd/              # 可执行入口
├── internal/         # 私有业务代码
├── pkg/              # 可被外部引用的库（谨慎）
├── api/              # proto / OpenAPI 等契约
├── configs/          # 配置模板
├── docs/             # 文档（本 Wiki 源文件）
└── scripts/          # CI / 本地脚本
```

## 命名与分层

- **包职责**：避免循环依赖；`internal` 不对外暴露。
- **日志**：结构化日志字段统一（`trace_id`、`user_id` 等）。

## 静态检查

列出必须通过的 CI 任务：`lint`、`unit test`、安全扫描（SAST/依赖漏洞）等。
