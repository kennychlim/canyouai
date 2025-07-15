<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# convert the markdown file to Chinese Traditional language text.

do not convert bash, ini, conf and yml file inside the ```bash|ini|conf|yml```

以下為將您提供的 Markdown 文件內容轉換為**繁體中文**的版本，並**保留所有 ```bash|ini|conf|yml```

# AI 系統安裝指南 - 為中小企業打造自託管 AI 堆疊

**作者**：Kenny Lim
**電子郵件**：kenny.chlim@gmail.com
**日期**：2025-07-12 21:05

## 前言

歡迎閱讀這份全面的教學，指導你如何打造一套**以隱私為核心、自託管的 AI 系統**，其效能可媲美 ChatGPT 等商業解決方案，但完全運行於你自己的硬體上。本指南能將一台普通電腦轉變為強大的 AI 工作站，支援大型語言模型（LLM）、圖像生成與智慧搜尋等功能。

這個專案靈感來自 LinkedIn「Can you AI?」的一篇貼文：「不只是做出華麗的簡報，而是要交付有潛力解決實際挑戰的運營解決方案！」本教學正是實踐這一理念——用最少的資源打造能應對複雜現實任務的 AI 解決方案。

## 為什麼這份教學重要

現今 AI 生態中，大多數人依賴於 ChatGPT、Claude 或 Gemini 等雲端服務。雖然方便，但這些服務有顯著限制：

- **隱私疑慮**：你的資料會被外部伺服器處理
- **成本累積**：每月訂閱費用快速增加
- **客製化有限**：只能使用預設模型和介面
- **需仰賴網路**：無法離線運作
- **資料主權**：敏感資料離開你的掌控

本教學將教你如何打造一套完全在自己硬體上運行的 AI 系統，讓你完全掌控資料，無限使用，且不需持續付費。

## 系統架構概覽

本 AI 系統設計為**模組化、自包含平台**，包括：

- **Ollama**：本地服務大型語言模型
- **Open WebUI**：類 ChatGPT 介面
- **SearXNG**：為 RAG（檢索增強生成）提供私人網頁搜尋
- **ComfyUI**：處理文字轉圖像生成
- **Nginx**：作為專業部署的反向代理

架構遵循安全最佳實踐，為每個服務建立專屬 Linux 使用者帳號，確保隔離與易於維護。

<p style="text-align:center;">每個服務的 Linux 使用者帳號</p>

## 硬體需求

本教學採用**一般人易取得的平價硬體**：

- **CPU**：Intel i5-2500S（4 核心，4 執行緒）
- **記憶體**：16GB RAM
- **儲存空間**：4TB SATA SSD
- **顯示卡**：2 張 NVIDIA RTX 2080 Ti（支援 NVLink）
- **總顯存**：44GB（每張 22GB）

雙 GPU 配置可最佳化資源分配：

- **GPU0**：專供 Ollama 執行 LLM
- **GPU1**：專供 ComfyUI 處理圖像生成


## 儲存配置策略

正確的儲存分配對 AI 工作負載至關重要。推薦的分割方案如下，確保模型與資料有足夠空間：

```bash
# Suggested partition allocation for 4TB drive:
# / (root): 1.6TB for system files
# /usr: 1TB for AI software installations  
# /var: 1TB for logs, databases, and user data
# swap: 16GB for memory overflow
```

完成所有模型下載與安裝後，實際儲存用量如下：

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


## 作業系統：Ubuntu Server 24.04 LTS 安裝

Ubuntu Server 24.04 LTS 是本 AI 系統的理想基礎，提供長期支援與優異硬體相容性。

### 安裝流程

請參考以下完整安裝指南：

- 主要指南：https://www.server-world.info/en/note?os=Ubuntu_24.04\&p=install
- 次要參考：「30 分鐘內安裝與配置 Ubuntu Server」by Ravi Saive，https://www.tecmint.com/ubuntu-server-setup/

**重要儲存配置說明**：安裝過程中特別注意儲存配置摘要。所有 AI 軟體都裝在 `/usr/local`，請分配足夠空間：

- **50%** 空間給 `/usr/local`（AI 應用）
- **30-40%** 給 `/var`（資料與日誌）
- **16GB** 給 swap


## GPU 驅動與 CUDA 安裝

現代 AI 工作負載需 GPU 加速。本節說明如何安裝 NVIDIA 驅動與 CUDA 工具包以獲得最佳效能。

### 硬體驗證

首先確認 NVIDIA 硬體正確偵測：

```bash
lspci | grep -i 'VGA compatible controller: NVIDIA'
```

雙 RTX 2080 Ti 預期輸出：

```
01:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1) 
```


### NVIDIA 驅動安裝

**重要**：建議使用 `ubuntu-drivers` 指令安裝最穩定：

```bash
# Check existing drivers
ubuntu-drivers devices

# Install recommended drivers
sudo ubuntu-drivers install

# Or install specific version
sudo apt install nvidia-driver-550
```

驗證安裝：

```bash
nvidia-driver-550/unknown,now 550.163.01-0ubuntu1 amd64 [installed]
cuda-toolkit-12-4/unknown,now 12.4.1-1 amd64 [installed]
```


### CUDA 工具包安裝

參考官方 NVIDIA 文件：https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html

**安裝指令：**

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt update
apt install cuda-toolkit-12-4
```

**手動 CUDA 安裝（替代方法）：**

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run -O cuda_12.4.0_550.54.14_linux.run
sudo sh cuda_12.4.0_550.54.14_linux.run --toolkit --silent

echo "export PATH=/usr/local/cuda-12.4/bin:\$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc
```

安裝並重啟後，驗證一切運作正常：

```bash
# Check NVIDIA driver
nvidia-smi

# Check CUDA installation  
nvcc --version
```

預期 nvidia-smi 輸出：

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

**NVLink 驗證：**

```bash
nvidia-smi topo -m
```

預期輸出：

```
        GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
GPU0     X      NV2     0-3     0               N/A
GPU1    NV2      X      0-3     0               N/A

Legend:
  X    = Self
  NV#  = Connection traversing a bonded set of # NVLinks
```

**CUDA 編譯器驗證：**

```bash
nvcc --version
```

預期輸出：

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Tue_Feb_27_16:19:38_PST_2024
Cuda compilation tools, release 12.4, V12.4.99
Build cuda_12.4.r12.4/compiler.33961263_0
```


## Ollama 安裝與配置

Ollama 是 LLM 基礎設施的核心，提供簡單 API 讓你在本地運行多種語言模型。

### 安裝

使用官方安裝腳本最可靠：

```bash
curl -fsSL https://ollama.com/install.sh | sh
```


### 服務配置

為 Ollama 建立專屬 systemd 服務：

**檔案：`/etc/systemd/system/ollama.service`**

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

Ollama 執行後，你可以下載與管理模型：

```bash
# List available models
ollama list

# Pull specific models
ollama pull deepseek-r1:8b-0528-qwen3-q4_K_M
ollama pull qwen2.5vl:7b
ollama pull snowflake-arctic-embed2:latest
```

**目前模型清單：**

```
NAME                                                              ID              SIZE      MODIFIED
deepseek-r1:8b-0528-qwen3-q8_0                                    cade62fd2850    8.9 GB    4 days ago
deepseek-r1:8b-0528-qwen3-q4_K_M                                  6995872bfe4c    5.2 GB    4 days ago
qwen2.5vl:7b                                                      5de93a84837d    6.0 GB    12 days ago
snowflake-arctic-embed2:latest                                    5de93a84837d    1.2 GB    12 days ago
```

**GPU0 總 VRAM 用量：21.3GB**

來源：https://github.com/ollama/ollama

## Python 3.11 安裝

本 AI 系統需 Python 3.11.13，效能最佳且與所需套件相容。

### 為什麼選擇 Python 3.11.13？

根據豐富商業與政府專案經驗，Python 3.11.13 是 2025 年 7 月最穩定可靠的 AI 專案版本，具備：

- 優異效能最佳化
- 與 AI/ML 函式庫強大相容性
- 長期支援與安全更新
- 生產環境證實的穩定性


### 安裝流程

**原始碼安裝網址**：https://www.python.org/downloads/release/python-31113/
**下載**：https://www.python.org/ftp/python/3.11.13/Python-3.11.13.tgz

**虛擬環境建立**：請參考官方文件：https://docs.python.org/3/library/venv.html

## Open WebUI 安裝

Open WebUI 提供進階的 ChatGPT 介面，讓你與本地 LLM 互動。

### 概覽

Open WebUI 是功能強大且豐富的網頁介面，提供：

- **ChatGPT 體驗**：使用者熟悉的介面
- **多模型支援**：可無縫切換不同 LLM
- **進階功能**：RAG、函式呼叫等
- **完整隱私**：所有運算皆在本地
- **WebSocket 支援**：即時互動


### 安裝步驟

**原始碼網址**：https://github.com/open-webui/open-webui

#### 步驟 1：建立專屬使用者帳號

```bash
# Create linux account with sudo privileges
sudo adduser --system --group --home /usr/local/openwebui openwebui
sudo usermod -aG sudo openwebui

# Set password for the account
sudo passwd openwebui
```


#### 步驟 2：安裝最佳化 Python

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


#### 步驟 3：建立虛擬環境

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


#### 步驟 4：安裝 Open WebUI 及相依套件

```bash
# Install core packages with specific versions for stability
pip install "open-webui==0.6.15" "chromadb==0.6.3" "packaging==23.2" "numpy==1.26.4" "googleapis-common-protos==1.63.2" "uvicorn==0.34.2" "pydantic==2.10.6" "onnxruntime==1.20.1"
```


### 配置

Open WebUI 預設於 8080 埠運作，並支援 WebSocket 即時功能。

**重要配置說明**：設置 `DATA_DIR=/var/OpenWebUI/data`，將使用者資料與應用程式分離。這樣可：

- 輕鬆每日備份
- 應用程式升級更乾淨
- 更佳的資料管理


### Nginx 配置

nginx 設定同時處理一般 HTTP 請求與 WebSocket 連線：

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

如需完整繁體中文翻譯，請告知是否需要繼續補全後續內容。

<div style="text-align: center">⁂</div>

[^1]: AI-System-Installation-Guide-Building-a-Self-Hosted-AI-Stack-for-Small-Business.md

