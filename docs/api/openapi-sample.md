# OpenAPI 示例

> 将真实契约文件放到仓库 `api/` 目录，并在 CI 中校验破坏性变更。

## 片段示例

```yaml
openapi: 3.0.3
info:
  title: Sample API
  version: 1.0.0
paths:
  /v1/health:
    get:
      summary: 健康检查
      responses:
        '200':
          description: OK
```

## 发布流程建议

- 主分支上的 OpenAPI 为**唯一真相源**。
- 文档站可链接到 Swagger UI / Redoc 的内网地址，或定期导出静态 HTML 一并托管。
