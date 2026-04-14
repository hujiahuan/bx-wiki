# 本地开发环境

## 前置依赖

- 操作系统：macOS / Linux / WSL2
- 运行时与构建工具：（按项目填写，如 Go 1.22+、Node 20+、Docker 等）
- 编辑器：推荐 VS Code / Cursor，并安装团队共享的 `settings.json` / 插件列表（链接到内部规范）。

## 一键初始化（示例）

```bash
git clone https://example.com/your-org/your-repo.git
cd your-repo
# 按项目补充：依赖安装、本地配置拷贝等
```

## 本地配置

- **环境变量**：从 `env.example` 复制为 `.env`（勿提交 `.env`）。
- **本地数据库**：可用 Docker Compose 启动，见仓库 `docker-compose.dev.yml`（若存在）。

## 常见问题

| 现象 | 处理 |
| --- | --- |
| 端口占用 | 修改本地端口或结束占用进程 |
| TLS 自签证书错误 | 开发环境导入根证书或关闭校验（仅本地） |
