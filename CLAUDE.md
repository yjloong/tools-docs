# CLAUDE.md — 文档仓库

基于 **MkDocs + Material 主题**的个人技术文档仓库。

## 定位

这是一个写给 AI 的参考指南仓库。每篇文档的结构是：
- **顶部**：一句话原因 + 功能说明 — 给人看
- **正文**：详细步骤、命令、踩坑记录 — 给 AI 参照执行

工具安装步骤不作为独立 skill，这里只记录**踩过的坑**和**离线环境特有操作**。

## 项目结构

```
tools-docs/
├── mkdocs.yml
├── docs/
│   ├── index.md
│   ├── template.md          # 文档模板
│   └── <分类>/index.md
└── CLAUDE.md
```

## 重点领域

- 离线环境搭建各类源（pip、apt、docker registry、helm repo 等）
- 工具从在线环境迁移到离线环境
- 架构迁移（x86 → arm 等）带来的兼容性问题

## 文档模板

```markdown
# <问题/场景标题>

> 原因：<为什么要做这件事>
> 功能：<做完能实现什么>

> 环境：<OS> | <工具版本> | <日期>

## 步骤

## 踩坑

## 参考
```
模板文件见 `docs/template.md`。

## 添加内容

1. 在 `docs/<分类>/` 下编辑对应 `index.md`
2. 新增分类时在 `mkdocs.yml` 的 `nav` 中注册
3. 部署：`mkdocs gh-deploy`
