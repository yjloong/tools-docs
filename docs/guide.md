# 文档编写指南

本文档是写给 AI 的编写规范。每次需要记录内容时，**严格按照此模板输出**。

## 目标读者

- **人**：只看标题、原因、功能，判断是否命中场景
- **AI**：参照正文执行操作、避开记录的坑

## 模板

```markdown
# <任务目标>

> 原因：<为什么要做这件事>
> 方案：<选了什么方案，为什么选它>
> 环境：<OS> | <工具名 版本> | <日期>

## 步骤

```bash
# 可复制执行的命令
```

## 踩坑

### 1. <坑名>

- **现象**：<错误信息或表现>
- **原因**：<根因>
- **解决**：<具体操作>
```

## 编写原则

1. **标题是任务目标，不是工具名** — `RAGFlow GPU 离线部署` 而非 `RAGFlow`
2. **方案一句话说清选型和理由** — 选 CUDA 13 镜像方案而非宿主机安装，因为目标机无 GPU 构建环境
3. **步骤是可复制的命令序列** — 粘贴到终端就能跑，不嵌入解释
4. **踩坑必须三联** — 现象→原因→解决，缺一不可
5. **不要废话** — 不写"首先、然后、接下来"，不写"值得注意的是"
6. **不要重复** — 步骤里写过的命令，踩坑里不再贴

## 示例

### 好

```markdown
# RAGFlow GPU 离线部署

> 原因：目标机有 H800 GPU，需要 GPU 加速文档解析
> 方案：自定义 Docker 镜像方案（含 PyTorch CUDA 13 + ONNX Runtime），不在目标机编译
> 环境：Ubuntu 22.04 | RAGFlow v0.25.5 | CUDA 13.0 | 2026-06-13

## 步骤

\```bash
docker build -t ragflow-server:v0.25.5-cu130 -f Dockerfile.gpu .
docker save ragflow-server:v0.25.5-cu130 | gzip > ragflow-gpu.tar.gz
\```

## 踩坑

### 1. ONNX Runtime GPU 不可用

- **现象**：`CUDAExecutionProvider not registered`
- **原因**：PyPI 上 `onnxruntime-gpu` 仅支持到 CUDA 12，CUDA 13 需从 nightly 源安装且必须 force-reinstall
- **解决**：`pip install --force-reinstall onnxruntime-gpu --index-url https://aiinfra.onnxruntime.ai/nightly`
```

### 差

```markdown
# RAGFlow

RAGFlow 是一个开源的 RAG 引擎。首先我们要安装 Docker，然后...

## 安装

apt install docker...

## 注意

要注意版本兼容性。
```

## 文件位置

项目根目录：`/home/ailab/tools_docs`

```
/home/ailab/tools_docs/
├── mkdocs.yml              # 导航配置
└── docs/
    ├── guide.md            # 本文档
    ├── template.md         # 模板
    ├── <分类>/             # 已存在的分类目录
    │   └── index.md
    └── <分类>/             # 新建分类目录
        └── <文件名>.md
```

### 放置规则

1. **新建文档** → `docs/<分类>/<文件名>.md`
   - 分类目录不存在则新建
   - 文件名用英文小写 + 连字符，如 `offline-pip-source.md`
2. **单篇追加** → 在已有分类的 `index.md` 中追加 `##` 二级标题段落
3. **新建分类** → 创建 `docs/<新分类>/index.md`，并在 `mkdocs.yml` 的 `nav` 中添加

### 注册导航

新增文档后必须更新 `/home/ailab/tools_docs/mkdocs.yml`：

```yaml
nav:
  - 首页: index.md
  - 已有分类: docker/index.md
  - 新分类: 新分类/index.md      # 添加这行
  - 已有分类:
      - 概览: docker/index.md
      - 具体文档: docker/xxx.md   # 子文档这样加
```

## 分类规则

- 内容涉及搭建离线源 → `pip/apt/docker/npm/...`
- 内容涉及环境迁移 → 按目标环境分类
- 无法归类的通用内容 → 放 `其他` 分类，后续再调整
