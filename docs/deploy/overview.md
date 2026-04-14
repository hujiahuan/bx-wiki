# 部署总览

## 交付形态

Docsify 为**纯静态站点**：`docs/` 目录内含 `index.html`、Markdown 与静态资源，无需 Node 运行时即可对外提供访问（构建机或本地预览才需要 `docsify-cli`）。

## 推荐部署方式

| 方式 | 适用 |
| --- | --- |
| Nginx / OpenResty 静态目录 | 内网虚拟机、裸金属 |
| 对象存储 + CDN | 大规模只读、多地域 |
| Kubernetes Ingress + 静态卷 | 与现有 K8s 体系统一 |

## 上线检查清单

- [ ] `index.html` 与所有相对路径资源可访问（注意大小写）。
- [ ] 若部署在子路径，已配置 `basePath`（见 [子路径部署](subpath.md)）。
- [ ] 已启用 HTTPS；内网亦建议 TLS。
- [ ] 访问控制（SSO / IP 白名单 / VPN）已按安全要求配置。

## 与 CI 集成（思路）

流水线将 `docs/` 同步到制品库或 rsync 到服务器；或使用 `aws s3 sync` / `coscmd` 等工具上传到对象存储。
