# 子路径部署（basePath）

当站点不是挂在域名根路径，而是形如 `https://intranet.example.com/wiki/` 时，需要在 `docs/index.html` 的 `$docsify` 中增加：

```javascript
window.$docsify = {
  basePath: '/wiki/',
  // ...其他配置
};
```

## 注意事项

- `basePath` 必须以 `/` 开头并以 `/` 结尾。
- Nginx 的 `location /wiki/` 需正确 alias 或 root，保证 `index.html` 与 `_sidebar.md` 等请求路径一致。
- 本地 `docsify serve` 预览时，如需模拟子路径，可使用 CLI 参数或反向代理，团队自行约定。

修改后务必在**预发环境**验证侧边栏与搜索跳转是否正常。
