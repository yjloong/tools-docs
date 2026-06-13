---
tags:
  - ragflow
  - gpu
  - cuda-13
  - nvidia-h800
  - docker
  - offline-deployment
  - onnx-runtime
  - pytorch-cu130
  - tiktoken
  - mkdocs
  - deepdoc
  - embedding
  - 20260613
---

# ragflow

> Last updated: 17:24:26

### Core Work

**RAGFlow GPU 离线部署方案（NVIDIA H800 + CUDA 13.0）**

- 从零调研 RAGFlow GPU 部署，确定架构：RAGFlow 容器用 GPU 做文档解析（DeepDoc/ONNX），推理通过 Ollama API 接入
- 发现 CUDA 13.0 兼容性问题：PyTorch 稳定版不支持，ONNX Runtime 需从微软 nightly 源安装，且必须 `--force-reinstall`（uv 检测已安装的 `onnxruntime-gpu` CUDA 12 版本会跳过替换）
- 构建自定义镜像 `ragflow-server:v0.25.5-cu130`，包含：
  - PyTorch 2.12.0+cu130 (nightly)
  - ONNX Runtime 1.26.0 + CUDAExecutionProvider
  - tiktoken offline fix（patch `load.py` 禁止联网，预置 `cl100k_base` 缓存）
  - cuDNN 9 / CUDA Toolkit 13 运行时库
- 踩坑：`onnxruntime-gpu-cuda13` 在 PyPI 上不存在（包名仍是 `onnxruntime-gpu`）
- 生成离线部署包 `~/ragflow-update-20260613.tar.gz` (6.3 GB)，含镜像 + compose 配置

### Supporting Work

- 修正 `docker-compose-gpu.yml` 中 MySQL `MYSQL_USER=root` 错误（不能与内置 root 重名）
- 修正 MinIO 凭据对齐（compose `ragflow`/`ragflow123` vs RAGFlow 期望 `rag_flow`/`infini_rag_flow`）
- Redis 添加密码 `infini_rag_flow` 解决 AUTH 错误
- 修正 nginx 配置缺失（v0.23.1 bug，切换到 v0.25.5）
- 宿主机 nginx 端口冲突处理（改 RAGFlow 到 8088）

### Key Decisions & Rationale

- **选 v0.25.5 而非 v0.23.1**：v0.23.1 缺少 nginx 配置 + pip 被 uv 替代导致 Dockerfile 构建失败
- **tiktoken patch 打进镜像而非 volume mount**：离线环境无法联网下载编码文件，commit 进镜像一劳永逸
- **ONNX Runtime CUDA 13 必须 `--force-reinstall`**：uv 检测到已安装的 CUDA 12 版本会跳过，导致 GPU 不可用
- **不在目标机上重建镜像**：目标机完全离线且无 Docker 构建环境，所有镜像在本机构建后打包传输
- **MinIO 卷删除后重建**：`MINIO_ROOT_USER/PASSWORD` 仅在首次创建时生效

### User Notes

- 用户有开发机（无 GPU）和目标机 `gwgpu001`（H800 80GB, CUDA 13.0, Docker 29.1.1, nvidia-ctk 已安装）
- 目标机已有 Ollama + vLLM 服务，显存占用 ~79GB，剩余 ~92GB
- 目标机完全离线，所有镜像和依赖需提前打包

### Version Log

1. 初始调研 — RAGFlow GPU 部署方案文档
2. v0.23.1 — 首次部署（废弃：nginx 配置缺失 + pip 不存在）
3. v0.25.5 — 切换版本，修正 MySQL/Redis/MinIO/tiktoken
4. v0.25.5-cu130 (ONNX RT 1.26.0 + tiktoken fix) — 最终可用镜像
