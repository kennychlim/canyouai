<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# AI 系统安装指南 - 为小型企业构建自托管 AI 堆栈

**作者**: Kenny Lim
**电子邮件**: kenny.chlim@gmail.com
**日期**: 2025-07-12 21:05

## 简介

欢迎来到这份全面的教程，教你如何构建一个**注重隐私、自托管的 AI 系统**，它能与 ChatGPT 等商业解决方案相媲美，但完全运行在你自己的硬件上。本指南将一台普通电脑转变为一台强大的 AI 工作站，能够运行大型语言模型 (LLMs)、生成图像并提供智能搜索功能。

“Can you AI?” 在领英上的一篇帖子启发了这个项目，其中写道：“不只是制作华丽的演示文稿，而是提供具有潜力解决实际挑战的运营解决方案！”本教程正是证明了这一点——用最少的资源创建了一个实用的 AI 解决方案，可以处理复杂的实际任务。

## 本教程为何重要

在当今的 AI 领域，大多数人依赖于像 ChatGPT、Claude 或 Gemini 这样的云服务。虽然方便，但这些服务存在显著的局限性：

- **隐私问题**：你的数据在外部服务器上处理
- **成本累积**：每月订阅费用迅速增加
- **定制受限**：你被限制在预构建的模型和界面中
- **依赖网络**：无法离线使用
- **数据主权**：你的敏感信息脱离了你的控制

本教程通过展示如何构建一个完全运行在你自己的硬件上的完整 AI 系统，解决了所有这些问题，让你完全掌控自己的数据，并无限制地使用，无需 recurring costs。

## 系统架构概览

该 AI 系统被设计为一个**模块化、独立的平台**，包括：

- **Ollama**：在本地服务大型语言模型
- **Open WebUI**：提供类似 ChatGPT 的界面
- **SearXNG**：为 RAG（检索增强生成）提供私人网页搜索功能
- **ComfyUI**：处理文本到图像的生成
- **Nginx**：作为生产部署的反向代理

该架构遵循安全最佳实践，为每个服务创建专用的 Linux 用户帐户，确保适当的隔离并简化维护。

<p style="text-align:center;">每个服务的 Linux 用户帐户</p>

## 硬件要求

本教程使用许多人已经拥有或可以廉价获得的**适度硬件**：

**CPU**：Intel i5-2500S (4 核, 4 线程)
**内存**：16GB RAM
**存储**：4TB SATA SSD
**显卡**：2块 NVIDIA RTX 2080 Ti 带 NVLink
**总显存**：44GB（每卡 22GB）

双 GPU 设置允许最佳资源分配：

- **GPU0**：专用于通过 Ollama 运行 LLM
- **GPU1**：通过 ComfyUI 处理图像生成


## 存储配置策略

正确的存储分配对于 AI 工作负载至关重要。推荐的分区方案确保为模型和数据提供足够的空间：

```bash
# Suggested partition allocation for 4TB drive:
# / (root): 1.6TB for system files
# /usr: 1TB for AI software installations  
# /var: 1TB for logs, databases, and user data
# swap: 16GB for memory overflow
```

完成所有模型下载的安装后，实际存储使用情况如下所示：

```bash
df -BG

Filesystem                                                                                   1G-blocks  Used Available Use% Mounted on
tmpfs                                                                                               2G    1G        2G   1% /run
/dev/mapper/ubuntu--vg-vg--root                                                                  1633G    7G     1543G   1% /
/dev/disk/by-id/dm-uuid-LVM-0KDedpPDV87PY2Secr2Snb270OK46qm43SssKR0ua32EH8rfLd4IvcyALX5pfEI6     1007G  496G      461G  52% /usr
tmpfs                                                                                               8G    0G        8G   0% /dev/shm
tmpfs                                                                                               1G    0G        1G   0% /run/lock
tmpfs                                                                                               8G    0G        8G   0% /run/shm
/dev/mapper/ubuntu--vg-vg--hm                                                                       1G    1G        1G   5% /home
/dev/sda2                                                                                           1G    1G        1G  27% /boot
/dev/mapper/ubuntu--vg-vg--var                                                                   1007G   15G      942G   2% /var
tmpfs                                                                                               2G    1G        2G   1% /run/user
```


## 操作系统：Ubuntu Server 24.04 LTS 安装

Ubuntu Server 24.04 LTS 为此 AI 系统提供了完美的基础，提供长期支持和出色的硬件兼容性。

### 安装过程

请遵循以下 Ubuntu Server 安装的全面指南：

- 主要指南：https://www.server-world.info/en/note?os=Ubuntu_24.04\&p=install
- 次要参考：“30 分钟内安装和配置 Ubuntu Server” by Ravi Saive at https://www.tecmint.com/ubuntu-server-setup/

**重要的存储配置说明**：在安装过程中，请特别注意存储配置摘要。由于所有 AI 软件都将安装到 `/usr/local`，请分配足够的空间：

- 总空间的 **50%** 分配给 `/usr/local` 用于 AI 应用程序
- **30-40%** 分配给 `/var` 用于数据和日志
- **16GB** 用于交换空间


## GPU 驱动和 CUDA 安装

现代 AI 工作负载需要适当的 GPU 加速。本节介绍安装 NVIDIA 驱动和 CUDA 工具包以获得最佳性能。

### 硬件验证

首先，验证你的 NVIDIA 硬件是否正确检测到：

```bash
lspci | grep -i 'VGA compatible controller: NVIDIA'
```

双 RTX 2080 Ti 设置的预期输出：

```
01:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1) 
```


### NVIDIA 驱动安装

**重要**: 使用 `ubuntu-drivers` 命令进行最稳定的安装：

```bash
# Check existing drivers
ubuntu-drivers devices

# Install recommended drivers
sudo ubuntu-drivers install

# Or install specific version
sudo apt install nvidia-driver-550
```

通过以下命令验证安装：

```bash
nvidia-driver-550/unknown,now 550.163.01-0ubuntu1 amd64 [installed]
cuda-toolkit-12-4/unknown,now 12.4.1-1 amd64 [installed]
```


### CUDA 工具包安装

参考 NVIDIA 官方文档：https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html

**安装命令：**

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt update
apt install cuda-toolkit-12-4
```

**手动 CUDA 安装（替代方法）：**

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run -O cuda_12.4.0_550.54.14_linux.run
sudo sh cuda_12.4.0_550.54.14_linux.run --toolkit --silent

echo "export PATH=/usr/local/cuda-12.4/bin:\$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc
```

安装并重启后，验证一切是否正常工作：

```bash
# Check NVIDIA driver
nvidia-smi

# Check CUDA installation  
nvcc --version
```

预期的 nvidia-smi 输出：

```
Sat Jul 12 12:25:22 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.163.01             Driver Version: 550.163.01     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 2080 Ti     Off |   00000000:01:00.0 Off |                  N/A |
| 32%   45C    P8             21W /  260W |       4MiB /  22528MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA GeForce RTX 2080 Ti     Off |   00000000:03:00.0 Off |                  N/A |
| 35%   47C    P8             32W /  260W |     158MiB /  22528MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```

**NVLink 验证：**

```bash
nvidia-smi topo -m
```

预期输出：

```
        GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
GPU0     X      NV2     0-3     0               N/A
GPU1    NV2      X      0-3     0               N/A

Legend:
  X    = Self
  NV#  = Connection traversing a bonded set of # NVLinks
```

**CUDA 编译器验证：**

```bash
nvcc --version
```

预期输出：

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Tue_Feb_27_16:19:38_PST_2024
Cuda compilation tools, release 12.4, V12.4.99
Build cuda_12.4.r12.4/compiler.33961263_0
```


## Ollama 安装和配置

Ollama 是我们 LLM 基础设施的支柱，为在本地运行各种语言模型提供了简单的 API。

### 安装

使用官方安装脚本进行最可靠的设置：

```bash
curl -fsSL https://ollama.com/install.sh | sh
```


### 服务配置

为 Ollama 创建一个专用的 systemd 服务：

**文件：`/etc/systemd/system/ollama.service`**

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
Environment="PATH=/usr/local/cuda-12.4/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin"
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```


### 模型管理

Ollama 运行后，你可以下载和管理模型：

```bash
# List available models
ollama list

# Pull specific models
ollama pull deepseek-r1:8b-0528-qwen3-q4_K_M
ollama pull qwen2.5vl:7b
ollama pull snowflake-arctic-embed2:latest
```

**当前模型清单：**

```
NAME                                                              ID              SIZE      MODIFIED
deepseek-r1:8b-0528-qwen3-q8_0                                    cade62fd2850    8.9 GB    4 days ago
deepseek-r1:8b-0528-qwen3-q4_K_M                                  6995872bfe4c    5.2 GB    4 days ago
qwen2.5vl:7b                                                      5de93a84837d    6.0 GB    12 days ago
snowflake-arctic-embed2:latest                                    5de93a84837d    1.2 GB    12 days ago
```

**GPU0 上的总 VRAM 使用量：21.3GB**

来源：https://github.com/ollama/ollama

## Python 3.11 安装

此 AI 系统需要 Python 3.11.13，它提供了最佳性能并与所有必需的软件包兼容。

### 为什么选择 Python 3.11.13？

根据丰富的商业和政府项目经验，Python 3.11.13 是截至 2025 年 7 月 AI 项目最稳定、最可靠的版本。它提供：

- 卓越的性能优化
- 与 AI/ML 库的强大兼容性
- 长期支持和安全更新
- 在生产环境中经过验证的稳定性


### 安装过程

**源代码安装 URL**：https://www.python.org/downloads/release/python-31113/
**下载**：https://www.python.org/ftp/python/3.11.13/Python-3.11.13.tgz

**虚拟环境创建**：遵循官方 Python 文档：https://docs.python.org/3/library/venv.html

## Open WebUI 安装

Open WebUI 提供了一个复杂的类似 ChatGPT 的界面，用于与你本地托管的语言模型进行交互。

### 概述

Open WebUI 是一个功能强大、功能丰富的 Web 界面，提供：

- **ChatGPT 般的体验**：熟悉的界面，方便用户使用
- **多模型支持**：无缝切换不同的 LLM
- **高级功能**：RAG 功能、函数调用等
- **完全隐私**：所有处理都在本地进行
- **WebSocket 支持**：实时通信以实现响应式交互


### 安装步骤

**源代码 URL**：https://github.com/open-webui/open-webui

#### 步骤 1：创建专用用户帐户

```bash
# Create linux account with sudo privileges
sudo adduser --system --group --home /usr/local/openwebui openwebui
sudo usermod -aG sudo openwebui

# Set password for the account
sudo passwd openwebui
```


#### 步骤 2：安装优化后的 Python

```bash
# Download Python source to /tmp
cd /tmp
wget https://www.python.org/ftp/python/3.11.13/Python-3.11.13.tgz
tar xzf Python-3.11.13.tgz
cd Python-3.11.13

# Configure with optimizations for OpenWebUI
./configure --prefix=/usr/local/openwebui/python --enable-optimizations --with-ensurepip=install

# Compile with parallel processing
make -j4
make install

# Add Python path to profile
echo 'export PATH="/usr/local/openwebui/python/bin:$PATH"' >> /usr/local/openwebui/.profile
```


#### 步骤 3：创建虚拟环境

```bash
# Switch to openwebui user
sudo -u openwebui bash

# Create virtual environment
python -m venv $HOME/owui

# Activate environment
source ~/owui/bin/activate

# Add activation to profile for persistence
echo "source ~/owui/bin/activate" >> ~/.profile
```


#### 步骤 4：安装 Open WebUI 和依赖项

```bash
# Install core packages with specific versions for stability
pip install "open-webui==0.6.15" "chromadb==0.6.3" "packaging==23.2" "numpy==1.26.4" "googleapis-common-protos==1.63.2" "uvicorn==0.34.2" "pydantic==2.10.6" "onnxruntime==1.20.1"
```


### 配置

Open WebUI 在 8080 端口上运行，支持 WebSocket 以实现实时功能。

**重要配置说明**：设置 `DATA_DIR=/var/OpenWebUI/data` 将用户数据与应用程序分离。这使得：

- 轻松进行日常备份
- 清洁的应用程序更新
- 更好的数据管理... \#\#\# Nginx 配置

nginx 配置处理常规 HTTP 请求和 WebSocket 连接：

```nginx
# Upstream configuration for OpenWebUI
upstream openwebui_backend {
   server 127.0.0.1:8080;
   keepalive 32;
   keepalive_requests 100;
   keepalive_timeout 60s;
}

location /ws {  # Match the WebSocket endpoint
   proxy_pass http://openwebui_backend;
   proxy_http_version 1.1;  # Required for WebSocket connections
   proxy_set_header Upgrade $http_upgrade;  # Handle upgrade requests
   proxy_set_header Connection "Upgrade";  # Set connection to upgrade
   proxy_set_header Host $host;  # Preserve the origi
```

<div style="text-align: center">⁂</div>

[^1]: AI-System-Installation-Guide-Building-a-Self-Hosted-AI-Stack-for-Small-Business.md

