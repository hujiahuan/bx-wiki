# HTTPS 与证书

## 证书来源

- 企业 PKI 签发的内网证书；
- 或 ACME（Let's Encrypt 等）自动续期（公网场景）。

## Nginx 基线

- 协议版本：TLS 1.2+（推荐 1.2 / 1.3）。
- 现代加密套件；关闭已知弱算法。
- 开启 `ssl_session_cache` 以提升握手性能。

## HSTS（公网可选）

仅在全站 HTTPS 且可长期维持时启用 `Strict-Transport-Security`，避免误配导致长时间无法访问。
