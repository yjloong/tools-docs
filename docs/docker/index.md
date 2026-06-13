# Docker

## Docker CE 安装 (Ubuntu 22.04)

> 原因：搭建容器化开发环境，部分项目需要离线运行 Docker
> 方案：安装 Docker CE 并配置免 sudo、离线镜像同步

> 环境：Ubuntu 22.04 | 2024-06-10

## 版本

| 组件 | 版本 | 备注 |
|------|------|------|
| Docker CE | 26.1.3 | 官方 apt 源 |
| containerd.io | 1.6.32 | Docker 依赖 |
| docker-compose-plugin | 2.27.1 | Docker Compose V2 |

## 步骤

```bash
# 卸载旧版本
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
    sudo apt-get remove $pkg
done

# 添加官方源
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 免 sudo 运行
sudo usermod -aG docker $USER
newgrp docker
```

### 踩坑

#### 1. `apt-get update` 时 GPG 报错

- **现象**：`NO_PUBKEY 8D81803C0EBFCD88`
- **原因**：Docker GPG 公钥未正确导入
- **解决**：手动 `curl` 下载 asc 文件并用 `signed-by` 指定

#### 2. `newgrp` 后仍然 `permission denied`

- **现象**：`docker ps` 报 `permission denied while trying to connect to the Docker daemon socket`
- **原因**：`newgrp` 只对当前 shell 生效，IDEA/VS Code 内的终端不共享 group
- **解决**：重启整个桌面会话，或 `sudo chmod 666 /var/run/docker.sock`（仅开发机）

### 参考链接

- [Docker 官方安装文档](https://docs.docker.com/engine/install/ubuntu/)
