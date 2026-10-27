# Gately 懒猫微服打包

[Gately](https://github.com/wangwanjie/gately) 的懒猫微服 LPK：开源网络网关工具集，DDNS、STUN 穿透、端口转发、反向代理、SSL 证书、WAF、Web 终端、网络诊断等，单文件约 22 MB，支持导入 Lucky 配置。

## 与上游部署的差异

- **不挂载 `/var/run/docker.sock`**：懒猫微服有自己的容器运行环境，Gately 的「Docker 管理」模块在微服上不可用；其余功能不受影响。需要完整 Docker 管理能力的用户请用 LightOS 或裸机部署上游镜像。
- **host 网络模式**：端口转发、STUN 穿透、Web 服务需要直接监听宿主机端口，与上游 compose 一致。
- **配置目录**：`/lzcapp/var/gately` → 容器内 `/etc/gately`（明文 JSON，`ssl.json` 含证书私钥，备份时注意安全）。

## 自动登录

参考懒猫官方 Jellyfin 应用的三阶段 inject 方案（`ctx.flow` → `ctx.persist` → 浏览器自动填充）：

| 阶段 | inject | 作用 |
| --- | --- | --- |
| request | `gately-capture-credentials` | 从 `POST /api/login`、`PUT /api/account` 的 JSON 里取出用户名与密码，暂存到 `ctx.flow` |
| response | `gately-commit-credentials` | 仅在 2xx 成功时写入 `ctx.persist` |
| response | `gately-forget-on-logout` | `/api/logout` 后清除记住的密码 |
| browser | `gately-login-autofill` | 在 `/#/login` 自动填充并提交（`autoSubmit: true`） |

初始密码来自安装向导的 `initial_password` 参数（默认随机 20 位），通过 `GATELY_INITIAL_PASSWORD` 传给容器，仅在 Gately 尚无账号时生效（上游 `internal/auth/auth.go` 的 `EnsureInitialized`）。

Gately 只保存 bcrypt 哈希、不存明文，所以明文必须在登录请求经过时捕获；用户之后在界面里改密，下次打开会自动使用新密码。

表单选择器用 Element Plus 渲染出的标准属性：

```yaml
userSelector: 'input[autocomplete="username"]'
passwordSelector: 'input[autocomplete="current-password"]'
```

退出登录会清除记住的密码，之后需手动输入一次。

## 发布

`.github/workflows/lazycat.yml` 使用 [lazycat-github-action](https://github.com/ca-x/lazycat-github-action) 自动构建并发布到双商店（喵喵商店私有 + 懒猫官方），镜像由 Action 转存到懒猫镜像源。
