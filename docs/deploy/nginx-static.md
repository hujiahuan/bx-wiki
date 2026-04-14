# 静态资源与 Nginx

以下示例将 **`docs/` 作为站点根目录** 部署到 `/var/www/bx-wiki`。路径请按实际修改。

## 最小配置

```nginx
server {
    listen 443 ssl http2;
    server_name wiki.internal.example.com;

    ssl_certificate     /etc/nginx/ssl/wiki.crt;
    ssl_certificate_key /etc/nginx/ssl/wiki.key;

    root /var/www/bx-wiki;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 缓存策略：HTML 短缓存，带 hash 的静态资源可长缓存（若后续引入打包）
    location = /index.html {
        add_header Cache-Control "no-cache";
    }
}
```

## SPA 路由说明

Docsify 使用 hash 路由（`/#/...`）时，通常**不需要**复杂 `try_files`；若你改为 history 模式（非默认），才需要将所有路径回落到 `index.html`。

## 日志

建议开启 `access_log` 与 `error_log`，日志脱敏（避免记录 query 中的 token）。
