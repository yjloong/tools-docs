# CLAUDE.md — 文档仓库

基于 **MkDocs + Material 主题**的个人技术文档仓库。

## 定位

每篇文档写给 AI 作为参考指南：
- **顶部**：原因 + 方案 — 给人看，判断是否命中场景
- **正文**：详细步骤、命令、踩坑记录 — 给 AI 参照执行

工具安装步骤不作为独立 skill，只记录**踩过的坑**和**离线环境特有操作**。

## 编写规范

**严格遵循 `docs/guide.md` 中的编写指南。** 核心规则：

1. 标题是任务目标，不是工具名
2. 方案一句话说清选型和理由
3. 步骤是可复制命令序列，不嵌入解释
4. 踩坑三联：现象 → 原因 → 解决
5. 不写废话，不重复

## 分类

- Docker、Kubernetes、Python、Git、数据库、CI/CD
- 新增分类时在 `mkdocs.yml` 的 `nav` 中注册

## 部署

```bash
mkdocs serve     # 本地预览
mkdocs gh-deploy # 发布到 GitHub Pages
```
