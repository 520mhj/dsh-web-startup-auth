# dsh-web-startup-auth (reverse-proxy compatible fork)

基于 [GDWhisper/dsh-web-startup-auth](https://github.com/GDWhisper/dsh-web-startup-auth) 修改，**兼容 Nginx/Caddy 反向代理场景**。

## 修改内容

原版插件在绑定 `127.0.0.1`（反代模式）时，仍会追溯包装所有已注册路由的 handler，将同步 handler 改为 async，导致 DSH 静态文件服务返回 **HTTP 400**。

本版本在 `bindHost !== '0.0.0.0'` 时**完全跳过路由包装**，仅保留前端修复脚本注入（`isLoopback` override + `crypto.randomUUID` polyfill），从而：

- 修复反代场景下的 HTTP 400
- 修复远程访问时 "settings are unavailable in this browser" 报错
- Nginx/Caddy 负责认证（Basic Auth）和 Host 伪装，插件负责前端环境修复

## 适用场景

- DSH 部署在服务器，通过 Nginx/Caddy 反代 + HTTPS 域名远程访问
- 反代层已做认证（Basic Auth / OAuth 等）
- DSH 本身绑定 `127.0.0.1:3080`

## 安装

```bash
# 从 GitHub 安装
dsh plugin --profile web add https://github.com/你的用户名/dsh-web-startup-auth.git

# 或从本地目录安装
dsh plugin --profile web add /path/to/dsh-web-startup-auth

# 重启 DSH
pm2 restart dsh-web
```

## Nginx 反代参考配置

```nginx
location / {
    proxy_pass http://127.0.0.1:3080;

    # 伪装本地访问，解决特权端点 403
    proxy_set_header Host 127.0.0.1:3080;
    proxy_set_header Origin http://127.0.0.1:3080;
    proxy_set_header Referer http://127.0.0.1:3080/;
    proxy_set_header Sec-Fetch-Site "";
    proxy_set_header Sec-Fetch-Mode "";
    proxy_set_header Sec-Fetch-Dest "";
    proxy_set_header Sec-Fetch-User "";

    # WebSocket
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_read_timeout 86400;
    proxy_send_timeout 86400;
    proxy_buffering off;
}
```

## 直接暴露模式（0.0.0.0）

如果需要直接绑定 `0.0.0.0` 暴露到公网（不推荐），插件仍保留原版的用户名密码认证功能，启动时加 `--host 0.0.0.0` 即可。

## License

MIT — 同原项目
