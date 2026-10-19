# Gately 懒猫微服打包

[Gately](https://github.com/wangwanjie/gately) 的懒猫微服 LPK：开源网络网关工具集，DDNS、STUN 穿透、端口转发、反向代理、SSL 证书、WAF、Web 终端、网络诊断等，单文件约 22 MB，支持导入 Lucky 配置。

## 与上游部署的差异

- **不挂载 `/var/run/docker.sock`**：懒猫微服有自己的容器运行环境，Gately 的「Docker 管理」模块在微服上不可用；其余功能不受影响。需要完整 Docker 管理能力的用户请用 LightOS 或裸机部署上游镜像。
- **host 网络模式**：端口转发、STUN 穿透、Web 服务需要直接监听宿主机端口，与上游 compose 一致。
- **配置目录**：`/lzcapp/var/gately` → 容器内 `/etc/gately`（明文 JSON，`ssl.json` 含证书私钥，备份时注意安全）。

## 首次登录

安装完成后打开应用域名，初始账号 `admin`，初始密码在应用日志里：

```sh
lzc-cli project log -s gately --tail 200 | grep -i "initial password"
```

或在懒猫微服管理界面查看容器日志，搜索 `initial password`。登录后请立即在「设置」中修改密码并开启两步验证。

## 发布

`.github/workflows/lazycat.yml` 使用 [lazycat-github-action](https://github.com/ca-x/lazycat-github-action) 自动构建并发布到双商店（喵喵商店私有 + 懒猫官方），镜像由 Action 转存到懒猫镜像源。
