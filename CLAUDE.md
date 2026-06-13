# CLAUDE.md — 工具安装文档

基于 **MkDocs + Material 主题**的工具安装文档仓库。记录各工具的安装方法、踩坑记录和对应版本。

## 项目结构

```
tools-docs/
├── mkdocs.yml              # MkDocs 配置
├── docs/                   # 所有 markdown 文档
│   ├── index.md            # 首页（也是 README.md）
│   ├── docker/
│   │   └── index.md
│   ├── kubernetes/
│   │   └── index.md
│   └── python/
│       └── index.md
└── CLAUDE.md
```

## 添加新工具文档

1. 在 `docs/<分类>/` 下创建目录和 `index.md`
2. 在 `mkdocs.yml` 的 `nav` 中注册
3. 文档使用统一模板（见下文）

## 文档模板

每个工具文档遵循固定结构：

```markdown
# <工具名> 安装记录

> 环境：<OS 版本> | <工具版本> | <日期>

## 安装步骤
\```bash
# 安装命令
\```

## 踩坑

### 1. <问题标题>
- **现象**：<错误信息或表现>
- **原因**：<根本原因>
- **解决**：<具体解决步骤>

## 参考链接
- [xxx](url)
```

## MkDocs 命令

```bash
mkdocs serve     # 本地预览 http://localhost:8000
mkdocs build     # 构建静态站点
mkdocs gh-deploy # 部署到 GitHub Pages
```
