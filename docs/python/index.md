# Python

## pyenv + Poetry 开发环境搭建 (Ubuntu 22.04)

> 原因：需要多版本 Python 共存，统一包管理，未来可能迁移到离线 pip 源
> 方案：pyenv 管理 Python 版本，Poetry 管理项目依赖

> 环境：Ubuntu 22.04 | 2024-05-28

## 版本

| 组件 | 版本 | 备注 |
|------|------|------|
| pyenv | 2.4.1 | curl 安装 |
| Python | 3.12.3 | pyenv 管理 |
| Poetry | 1.8.3 | 官方安装脚本 |

## 步骤

```bash
# pyenv
curl https://pyenv.run | bash

# 添加到 ~/.bashrc
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
exec "$SHELL"

# 安装 Python 编译依赖
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# 安装指定 Python 版本
pyenv install 3.12.3
pyenv global 3.12.3

# Poetry
curl -sSL https://install.python-poetry.org | python3 -

echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

### 踩坑

#### 1. `pyenv install` 编译失败

- **现象**：`BUILD FAILED (Ubuntu 22.04 using python-build ...)`
- **原因**：缺少 `libssl-dev` 和 `libbz2-dev` → `ssl` 和 `bz2` 模块编译失败
- **解决**：先安装完整编译依赖（见上方 apt 命令）

#### 2. `poetry install` 报 `Incompatible model version`

- **现象**：`The lock file is not compatible with the current version of Poetry`
- **原因**：团队内 Poetry 版本不一致，lock 文件由高版本生成
- **解决**：`poetry self update` 升级到最新，或 `poetry lock --no-update` 重新生成

### 参考链接

- [pyenv 官方](https://github.com/pyenv/pyenv)
- [Poetry 官方](https://python-poetry.org/docs/)
