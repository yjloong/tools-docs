# RAGFlow GPU 离线部署

> 原因：目标机 H800 + CUDA 13.0，需 GPU 加速文档解析（DeepDoc/ONNX）+ 接入已有 Ollama 推理
> 方案：自定义 Docker 镜像（PyTorch cu130 + ONNX Runtime CUDA 13 + tiktoken 离线），本机构建打包传输，不在目标机编译
> 环境：Ubuntu 22.04 | RAGFlow v0.25.5 | CUDA 13.0 | Docker 29.1+ | 2026-06-13

## 架构

```
目标机 gwgpu001 (H800 80GB, CUDA 13.0)
┌─────────────────────────────────────────────┐
│  RAGFlow (Docker)          Ollama (宿主机)    │
│  ragflow-server            端口 11434         │
│  ├─ DeepDoc OCR → GPU      ├─ Chat LLM       │
│  ├─ API Server :9380        └─ Embedding     │
│  └─ Nginx :80 → WebUI                        │
│                                               │
│  MySQL :5455 | ES :1200 | Redis | MinIO      │
└─────────────────────────────────────────────┘
```

## 前置条件

目标机：
- Docker >= 24.0、NVIDIA Container Toolkit 已安装并配置
- `vm.max_map_count >= 262144`

验证：

```bash
nvidia-smi
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu24.04 nvidia-smi
sysctl vm.max_map_count
```

## 步骤

### 1. 构建自定义镜像（开发机，有网络）

```bash
cd /home/ailab/ragflow
docker build --no-cache -t ragflow-server:v0.25.5-cu130 -f Dockerfile.cuda13-custom .
```

关键文件 `Dockerfile.cuda13-custom`：

```dockerfile
FROM infiniflow/ragflow:v0.25.5

# 卸载 CUDA 12 包
RUN uv pip uninstall -y torch torchvision torchaudio onnxruntime-gpu onnxruntime 2>/dev/null || true

# PyTorch CUDA 13 (nightly)
RUN uv pip install --pre torch torchvision torchaudio \
    --index-url https://download.pytorch.org/whl/nightly/cu130

# ONNX Runtime CUDA 13（必须 force-reinstall，否则 uv 跳过）
RUN uv pip install --force-reinstall --pre --index-url \
    https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/ort-cuda-13-nightly/pypi/simple/ \
    onnxruntime-gpu

# tiktoken 离线修复
COPY tiktoken-cache /tmp/data-gym-cache
COPY tiktoken_preload.py /tmp/tiktoken_preload.py
RUN python3 /tmp/tiktoken_preload.py

ENV DEVICE=gpu
```

### 2. 验证镜像

```bash
docker run --rm --entrypoint bash ragflow-server:v0.25.5-cu130 -c "
python3 -c 'import torch; print(torch.__version__)'
python3 -c 'import onnxruntime as ort; print(ort.__version__, ort.get_available_providers())'
python3 -c 'import tiktoken; enc = tiktoken.get_encoding(\"cl100k_base\"); print(enc.name)'
ldd /ragflow/.venv/lib/python3.13/site-packages/onnxruntime/capi/onnxruntime_providers_cuda.so | grep 'not found' || echo 'OK'
"
```

期望输出：`CUDAExecutionProvider`、`cl100k_base`、`OK: no missing libs`

### 3. 导出更新包

```bash
mkdir -p ~/ragflow-update-$(date +%Y%m%d)/images
docker save -o ~/ragflow-update-$(date +%Y%m%d)/images/ragflow-server_v0.25.5-cu130.tar ragflow-server:v0.25.5-cu130
cp /home/ailab/ragflow/docker/docker-compose-gpu.yml ~/ragflow-update-$(date +%Y%m%d)/
cp /home/ailab/ragflow/docker/.env.gpu ~/ragflow-update-$(date +%Y%m%d)/
cd ~ && tar -czf ragflow-update-$(date +%Y%m%d).tar.gz ragflow-update-$(date +%Y%m%d)
```

### 4. 传输到目标机

```bash
scp ~/ragflow-update-*.tar.gz yangjl@gwgpu001:~/
```

### 5. 目标机首次部署（全量包）

```bash
tar -xzf ~/ragflow-offline-*.tar.gz -C ~/ragflow/
cd ~/ragflow/ragflow-offline-*
bash deploy-offline.sh
```

### 6. 目标机更新（仅 ragflow-server 变更）

```bash
cd ~/ragflow/ragflow-offline-*
docker compose -f docker-compose-gpu.yml down
tar -xzf ~/ragflow-update-*.tar.gz -C ~/ragflow/ --strip-components=1
docker load -i ~/ragflow/images/ragflow-server_v0.25.5-cu130.tar
docker compose -f docker-compose-gpu.yml up -d
```

### 7. 配置模型（WebUI）

浏览器打开 `http://<目标机IP>:8088`：

- 设置 → 模型提供商 → 添加 Ollama
- Base URL: `http://host.docker.internal:11434`
- Chat 模型: 已有的 Ollama chat 模型名
- Embedding 模型: `bge-m3`

## docker-compose-gpu.yml 关键修正

```yaml
# MySQL: 不能设 MYSQL_USER=root（与内置 root 冲突）
mysql:
  environment:
    MYSQL_ROOT_PASSWORD: infini_rag_flow
    MYSQL_DATABASE: rag_flow
    # 不要加 MYSQL_USER / MYSQL_PASSWORD

# Redis: 需要密码，否则 AUTH 报错
redis:
  command: redis-server --requirepass infini_rag_flow

# MinIO: 凭据必须与 RAGFlow service_conf.yaml 一致
minio:
  environment:
    MINIO_ROOT_USER: rag_flow
    MINIO_ROOT_PASSWORD: infini_rag_flow

# RAGFlow: GPU device reservation
ragflow-server:
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            count: all
            capabilities: [gpu]
  environment:
    - DEVICE=gpu
    - REDIS_PASSWORD=infini_rag_flow
```

## 踩坑

### 1. ONNX Runtime GPU 不可用

- **现象**：`libcublasLt.so.12: cannot open shared object file` → 回退 CPU
- **原因**：PyPI 上 `onnxruntime-gpu` 链接的是 CUDA 12 的 `libcublasLt.so.12`，CUDA 13 系统只有 `.so.13`。包名 `onnxruntime-gpu-cuda13` 在 PyPI 上不存在
- **解决**：从微软 nightly 源安装，必须 `--force-reinstall`，否则 uv 认为已安装 CUDA 12 版本而跳过

```bash
uv pip install --force-reinstall --pre --index-url \
  https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/ort-cuda-13-nightly/pypi/simple/ \
  onnxruntime-gpu
```

### 2. tiktoken 离线环境加载失败

- **现象**：`Failed to resolve 'openaipublic.blob.core.windows.net'` → 启动报错
- **原因**：`tiktoken.get_encoding("cl100k_base")` 首次调用时联网下载 BPE 编码文件
- **解决**：预下载缓存文件到 `/tmp/data-gym-cache/<sha1-of-url>`，patch `load.py` 中 `read_file()` 的 HTTP 分支改为读本地缓存、`check_hash()` 跳过校验

### 3. MySQL `MYSQL_USER=root` 启动失败

- **现象**：`MYSQL_USER="root", MYSQL_USER and MYSQL_PASSWORD are for configuring a regular user and cannot be used for the root user`
- **原因**：MySQL 官方镜像规定 `MYSQL_USER` 只能创建普通用户，不能设为 root
- **解决**：删除 `MYSQL_USER`/`MYSQL_PASSWORD`，只保留 `MYSQL_ROOT_PASSWORD`

### 4. MinIO 凭据不匹配

- **现象**：`InvalidAccessKeyId: The Access Key Id you provided does not exist in our records`
- **原因**：`docker-compose-gpu.yml` 写的是 `ragflow`/`ragflow123`，但 RAGFlow 的 `service_conf.yaml` 期望 `rag_flow`/`infini_rag_flow`
- **解决**：对齐 compose 文件中的 `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD`

### 5. Redis AUTH 报错

- **现象**：`valkey.exceptions.AuthenticationError: AUTH <password> called without any password configured`
- **原因**：RAGFlow 使用的 valkey 客户端默认尝试 AUTH，但 Redis 容器没有设置密码
- **解决**：`redis-server --requirepass infini_rag_flow`，RAGFlow 侧传 `REDIS_PASSWORD`

### 6. Nginx 返回默认页，非 WebUI

- **现象**：浏览器看到 `Welcome to nginx!` 而非 RAGFlow 登录页
- **原因**：v0.23.1 镜像缺少 RAGFlow 的 nginx 配置（`/etc/nginx/conf.d/ragflow.conf.python`）；宿主机 nginx 占用了 80 端口
- **解决**：切换到 v0.25.5（自带完整 nginx 配置）；改 `NGINX_HTTP_PORT=8088` 避开宿主机端口冲突

### 7. Dockerfile 构建时 `pip: command not found`

- **现象**：`/bin/bash: line 1: pip: command not found`
- **原因**：RAGFlow v0.23.1+ 镜像用 `uv` 替代了 `pip`
- **解决**：所有 `pip` 命令改为 `uv pip`

## 版本记录

| 日期 | 镜像 | 变更 |
|------|------|------|
| 2026-06-12 | `v0.23.1-cu130` | 首次构建（废弃：nginx 缺失） |
| 2026-06-12 | `v0.25.5-cu130` (ONNX RT 1.23.2) | 切换版本，修正 MySQL/Redis/MinIO |
| 2026-06-13 | `v0.25.5-cu130` (ONNX RT 1.26.0 + tiktoken) | 修正 ONNX RT 安装方式 + tiktoken patch |
