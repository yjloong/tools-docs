# CLAUDE.md — 技能库

基于 **MkDocs + Material 主题**的个人技能库，定位类似 Claude Code skill：
每篇文档对应一个具体**问题场景**或**工具安装方案**，可复现、可独立使用。

## 项目结构

```
tools-docs/
├── mkdocs.yml              # MkDocs 配置
├── docs/                   # 所有文档
│   ├── index.md            # 站点首页
│   ├── template.md         # 文档模板
│   └── <分类>/index.md     # 分类文档
└── CLAUDE.md
```

## 文档编写原则

- **单篇自洽**：每篇不依赖其它文档，独立可用
- **问题驱动**：标题是问题或场景，而非工具名
- **信息密集**：环境 + 命令 + 坑 + 链接，没有废话
- **版本锚定**：每条记录标上版本号，方便回溯

## 文档模板

```markdown
# <问题 / 场景标题>

> 环境：<OS> | <工具名> <版本号> | <YYYY-MM-DD>

## 场景

## 解决步骤

## 踩坑

## 参考链接
```

## 添加新文档

1. 在 `docs/<分类>/` 下编辑对应 `index.md`
2. 若新增分类，在 `mkdocs.yml` 的 `nav` 中注册
3. 部署：`mkdocs gh-deploy`

## 常用命令

```bash
mkdocs serve     # 本地预览
mkdocs gh-deploy # 部署到 GitHub Pages
```
