# 认证与鉴权

## 方式概览

| 场景 | 方案 | 说明 |
| --- | --- | --- |
| 用户登录 | OAuth2 / OIDC / 自建 JWT | 刷新令牌轮换策略 |
| 服务间 | mTLS / 服务账号 Token | 最小权限 |

## 请求头（示例）

```http
Authorization: Bearer <access_token>
```

## Token 生命周期

- **Access Token**：短有效期（如 15 分钟）。
- **Refresh Token**：长有效期 + 可撤销；存储策略（HttpOnly Cookie 或安全存储）由客户端类型决定。

## 权限模型

简述 RBAC / ABAC 规则与典型角色：`admin`、`operator`、`viewer`。
