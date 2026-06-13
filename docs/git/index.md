# Git

## 常用配置与技巧

> 原因：多环境多账号 Git 操作经常遇到认证和配置问题
> 方案：统一 Git 基础配置、SSH 密钥管理、常见报错修复

> 环境：Ubuntu 22.04 | 2024-06-01

## 版本

| 组件 | 版本 | 备注 |
|------|------|------|
| Git | 2.43.0 | 系统仓库 |
| OpenSSH | 9.6p1 | 系统仓库 |

## 基础配置

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 换行符统一 LF（Linux/Mac 协作）
git config --global core.autocrlf input

# 默认分支
git config --global init.defaultBranch main

# 常用别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all -20"
```

### SSH 密钥

```bash
ssh-keygen -t ed25519 -C "your@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 复制公钥添加到 GitHub Settings → SSH Keys
cat ~/.ssh/id_ed25519.pub
```

### 踩坑

#### 1. `git push` 提示 `Permission denied (publickey)`

- **现象**：`git@github.com: Permission denied (publickey)`
- **原因**：SSH agent 未加载密钥，或 GitHub 上没有对应公钥
- **解决**：`ssh-add ~/.ssh/id_ed25519` 并确认 GitHub 已添加公钥

#### 2. `git commit` 后日期显示错误

- **现象**：commit 后 GitHub 显示时间为 1970 年
- **原因**：本机时间同步未开启
- **解决**：`sudo timedatectl set-ntp true`

### 参考链接

- [Git 官方文档](https://git-scm.com/doc)
- [GitHub SSH 配置](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
