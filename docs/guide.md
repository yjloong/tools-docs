# 文档编写指南

本文档是写给 AI 的编写规范。每次记录内容时 **严格按照此模板输出**。

## 核心目标

**文档的目标是让读者完整复现。** 每篇文档必须包含足够的细节：精确版本号、完整步骤序列、以及过程中遇到的每一个障碍。读者按步骤操作就应该得到相同结果，不需要自行推断任何信息。

## 目标读者

- **人**：看标题、原因、方案，判断是否命中场景
- **AI**：按步骤执行，照版本锁定依赖，参照注意要点避坑

## 模板

```markdown
# <任务目标>

> 原因：<为什么要做这件事>
> 方案：<选了什么方案，为什么选它>
> 环境：<OS 版本> | <日期>

## 版本

| 组件 | 版本 | 备注 |
|------|------|------|
| <组件1> | <x.y.z> | <来源/安装方式> |
| <组件2> | <x.y.z> | |

## 步骤

### 1. <阶段性标题>

```bash
# 可复制执行的命令
```

### 2. <阶段性标题>

...

## 注意

- <要点1>：<为什么重要，忽略会导致什么问题>
- <要点2>：<适用条件和限制>

## 踩坑

### 1. <坑名>

- **现象**：<完整错误信息>
- **原因**：<根因分析>
- **解决**：<具体操作步骤>
```

## 编写规则

### 核心约束（必须遵守）

1. **版本必须精确到小版本号** — `ONNX Runtime 1.26.0` 而非 `ONNX Runtime 1.x`。版本表列出所有参与组件，不遗漏
2. **步骤从零到验证** — 覆盖前置准备、安装、配置、启动、验证五阶段，不跳过任何一步
3. **命令直接可复制** — 每条命令包含完整路径和参数，不写 `<placeholder>`
4. **注意 ≠ 踩坑** — 「注意」是事前提醒（不看会出错），「踩坑」是事后记录（实际撞过的错）
5. **现象给完整报错** — 不概括，贴原始错误信息，方便搜索

### 写法约束

6. **标题是任务目标** — `pip 离线源搭建` 而非 `pip`
7. **方案一句话带理由** — 说明选了什么以及为什么
8. **不写废话** — 不用"首先然后接下来值得注意的是"
9. **不重复** — 步骤里写过的命令，后面不再贴
10. **分类归档** — 内容放对目录，不确定的先放最相关的

## 示例（好的）

```markdown
# pip 离线源搭建

> 原因：目标服务器完全离线，需要本地 pip 源安装 Python 包
> 方案：用 pip2pi 在 devpi-server 上搭建本地 PyPI 镜像，pip 指向本地源安装
> 环境：Ubuntu 24.04 | 2026-06-13

## 版本

| 组件 | 版本 | 备注 |
|------|------|------|
| devpi-server | 6.9.2 | pip install |
| devpi-web | 3.6.1 |  Web 界面 |
| pip2pi | 0.8.2 | 离线包同步 |
| Python | 3.12.3 | 系统安装 |

## 步骤

### 1. 安装 devpi

\```bash
pip install devpi-server==6.9.2 devpi-web==3.6.1
devpi-init --serverdir ~/devpi
devpi-server --serverdir ~/devpi --host 0.0.0.0 --port 3141
\```

### 2. 配置 pip 指向本地源

\```bash
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = http://192.168.10.20:3141/root/pypi/+simple/
trusted-host = 192.168.10.20
EOF
\```

### 3. 从在线机同步包到离线机

\```bash
# 在线机：下载包
pip download -d /tmp/packages/ -r requirements.txt
# 拷贝到离线机
scp -r /tmp/packages/ user@offline:/tmp/packages/
# 离线机：上传到 devpi
devpi upload --from-dir /tmp/packages/
\```

### 4. 验证

\```bash
pip install --no-cache-dir numpy==1.26.4
python -c "import numpy; print(numpy.__version__)"
\```

## 注意

- devpi 默认监听 localhost，离线环境需改 `--host 0.0.0.0`
- 首次 `devpi-init` 后需要创建用户和索引：`devpi use http://localhost:3141 && devpi user -c root password=xxx && devpi login root --password=xxx && devpi index -c prod`
- pip.conf 中的 trusted-host 在离线环境必须配置，否则 SSL 验证失败

## 踩坑

### 1. devpi upload 报 403

- **现象**：`403 Forbidden: You are not allowed to upload to root/pypi`
- **原因**：`root/pypi` 是只读镜像索引，不能直接上传
- **解决**：`devpi index -c prod` 创建可写索引，pip 指向 `root/prod/+simple/`
```

## 示例（差的）

```markdown
# Python 离线环境搭建

## 安装

装 devpi，然后配一下 pip 就行。

## 注意

注意网络环境。
```

## 文件位置

项目根目录：`/home/ailab/tools_docs`

```
/home/ailab/tools_docs/
├── mkdocs.yml
├── docs/
│   ├── guide.md
│   ├── template.md
│   └── <分类>/<文件名>.md
```

### 放置规则

1. **新建文档** → `docs/<分类>/<文件名>.md`（文件名英文小写+连字符）
2. **追加到已有分类** → 在 `index.md` 末尾追加 `##` 段落
3. **新增分类** → 创建 `docs/<新分类>/index.md`，更新 `mkdocs.yml` 的 `nav`

### 注册导航

```yaml
nav:
  - 已有分类: docker/index.md
  - 新分类: 新分类/index.md    # 添加这行
```

## 分类目录

| 分类 | 目录 | 典型内容 |
|------|------|---------|
| Docker | `docker/` | 安装、镜像同步、GPU 容器 |
| Kubernetes | `kubernetes/` | 离线部署、源替换 |
| Python | `python/` | pip 离线源、环境迁移 |
| Git | `git/` | 配置、常见报错 |
| 数据库 | `database/` | 安装、迁移 |
| CI/CD | `cicd/` | 离线流水线、制品管理 |
