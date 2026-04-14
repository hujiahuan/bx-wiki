# 接口约定

## 基础信息

| 项 | 约定 |
| --- | --- |
| Base URL | `https://api.example.com`（按环境替换） |
| 协议 | HTTPS |
| 数据格式 | `Content-Type: application/json; charset=utf-8` |
| 时间 | ISO 8601 UTC，如 `2026-04-11T08:00:00Z` |

## 版本化

推荐使用 URL 前缀版本，例如：`/v1/resource`。

## 幂等与重试

- 写操作建议支持 **Idempotency-Key** 请求头（若已实现）。
- 客户端对 `429` / `503` 使用指数退避重试，并设置最大重试次数。

## 分页（示例）

```http
GET /v1/items?page=2&page_size=50
```

响应体字段名与团队统一：`items`、`total`、`page`、`page_size`。
