# GitHub Actions 自动部署

推送到 `main` 后，GitHub Actions 会构建 Linux/amd64 镜像、推送到阿里云私有仓库，并通过 SSH 更新服务器上的 `start-page` 容器。镜像使用 Git 提交 SHA 作为不可变版本标签，同时更新 `latest`。部署失败时会尝试恢复更新前的容器镜像。

## 1. 准备服务器

- 安装 Docker，并确认部署用户可以运行 `docker` 命令（通常需要加入 `docker` 用户组）。
- 配置 SSH 公钥登录：将私钥对应的**公钥**加入服务器部署用户的 `~/.ssh/authorized_keys`。
- 在云服务器安全组和系统防火墙中允许 SSH 端口，以及对外提供服务的 `18080/tcp`。
- DNS 将 `start.zhengjiao.cc` 指向服务器公网 IP；如使用 Nginx/Caddy 反向代理，可将域名转发到 `127.0.0.1:18080`。

## 2. 设置 GitHub Actions Secrets

在 GitHub 仓库打开 **Settings → Secrets and variables → Actions → New repository secret**，添加：

| Secret 名称 | 值 |
| --- | --- |
| `ALIYUN_REGISTRY_USERNAME` | 阿里云镜像仓库用户名，建议使用有该仓库推送权限的 RAM 用户 |
| `ALIYUN_REGISTRY_PASSWORD` | 对应的阿里云镜像仓库访问凭证密码 |
| `DEPLOY_HOST` | 服务器公网 IP 或 SSH 可连接的主机名 |
| `DEPLOY_USER` | 服务器 SSH 用户名 |
| `DEPLOY_SSH_KEY` | SSH 私钥完整内容，包含 BEGIN/END 行 |
| `DEPLOY_PORT` | SSH 端口；若使用默认端口 22，可不设置 |

不要把密码、私钥或其他凭证提交到仓库。私有镜像仓库建议为 CI 推送和服务器拉取创建独立的 RAM 凭证，并仅授予所需权限。

## 3. 部署

先把本次代码和工作流推送到 GitHub 的 `main` 分支。之后每次推送到 `main` 都会自动部署；也可以在仓库的 **Actions → Build and deploy → Run workflow** 手动触发。

服务器上的站点端口为 `18080`。回退时可以在服务器使用之前部署记录里的提交 SHA 镜像重新创建容器。
