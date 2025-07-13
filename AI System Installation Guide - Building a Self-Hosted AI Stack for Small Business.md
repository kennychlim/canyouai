# AI System Installation Guide - Building a Self-Hosted AI Stack for Small Business

**Author**: Kenny Lim
**Email**: kenny.chlim@gmail.com
**Date**: 12-07-2025 21:05

## Introduction

Welcome to this comprehensive tutorial for building your own **privacy-focused, self-hosted AI system** that rivals commercial solutions like ChatGPT, but runs entirely on your own hardware. This guide transforms an ordinary PC into a powerful AI workstation capable of running Large Language Models (LLMs), generating images, and providing intelligent search capabilities.

A LinkedIn post from "Can you AI?" inspired this project with the message: "Not just build flashy presentations, but deliver operational solutions with the potential to solve real challenges!" This tutorial demonstrates exactly that - creating a practical AI solution with minimal resources that can handle complex real-world tasks.

## Why This Tutorial Matters

In today's AI landscape, most people depend on cloud-based services like ChatGPT, Claude, or Gemini. While convenient, these services come with significant limitations:

- **Privacy concerns**: Your data is processed on external servers
- **Cost accumulation**: Monthly subscriptions add up quickly
- **Limited customization**: You're restricted to pre-built models and interfaces
- **Internet dependency**: No offline capabilities
- **Data sovereignty**: Your sensitive information leaves your control

This tutorial addresses all these concerns by showing you how to build a complete AI system that runs entirely on your own hardware, giving you full control over your data and unlimited usage without recurring costs.

## System Architecture Overview

This AI system is designed as a **modular, self-contained platform** that includes:

- **Ollama**: Serves large language models locally
- **Open WebUI**: Provides a ChatGPT-like interface
- **SearXNG**: Offers private web search capabilities for RAG (Retrieval-Augmented Generation)
- **ComfyUI**: Handles text-to-image generation
- **Nginx**: Acts as a reverse proxy for professional deployment

The architecture follows security best practices by creating dedicated Linux user accounts for each service, ensuring proper isolation and making maintenance straightforward.

## Hardware Requirements

This tutorial uses **modest hardware** that many people already own or can acquire affordably:

**CPU**: Intel i5-2500S (4 cores, 4 threads)
**Memory**: 16GB RAM
**Storage**: 4TB SATA SSD
**Graphics**: 2× NVIDIA RTX 2080 Ti with NVLink
**Total VRAM**: 44GB (22GB per card)

The dual-GPU setup allows for optimal resource allocation:

- **GPU0**: Dedicated to running LLMs through Ollama
- **GPU1**: Handles image generation through ComfyUI


## Storage Configuration Strategy

Proper storage allocation is crucial for AI workloads. The recommended partition scheme ensures adequate space for models and data:

```bash
# Suggested partition allocation for 4TB drive:
# / (root): 1.6TB for system files
# /usr: 1TB for AI software installations  
# /var: 1TB for logs, databases, and user data
# swap: 16GB for memory overflow
```

After complete installation with all models downloaded, the actual storage usage appears as:

```bash
df -BG
```

```
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


## Operating System: Ubuntu Server 24.04 LTS Installation

Ubuntu Server 24.04 LTS provides the perfect foundation for this AI system, offering long-term support and excellent hardware compatibility.

### Installation Process

Follow these comprehensive guides for Ubuntu Server installation:

- Primary guide: https://www.server-world.info/en/note?os=Ubuntu_24.04\&p=install
- Secondary reference: "How to Install and Configure Ubuntu Server in 30 Minutes" by Ravi Saive at https://www.tecmint.com/ubuntu-server-setup/

**Critical Storage Configuration Note**: During installation, pay special attention to the storage configuration summary. Since all AI software will be installed to `/usr/local`, allocate sufficient space:

- **50%** of total space to `/usr/local` for AI applications
- **30-40%** to `/var` for data and logs
- **16GB** for swap space


## GPU Driver and CUDA Installation

Modern AI workloads require proper GPU acceleration. This section covers installing NVIDIA drivers and CUDA toolkit for optimal performance.

### Hardware Verification

First, verify your NVIDIA hardware is properly detected:

```bash
lspci | grep -i 'VGA compatible controller: NVIDIA'
```

Expected output for dual RTX 2080 Ti setup:

```
01:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1) 
```


### NVIDIA Driver Installation

**Important**: Use the `ubuntu-drivers` command for the most stable installation:

```bash
# Check existing drivers
ubuntu-drivers devices

# Install recommended drivers
sudo ubuntu-drivers install

# Or install specific version
sudo apt install nvidia-driver-550
```

Verify installation with:

```bash
nvidia-driver-550/unknown,now 550.163.01-0ubuntu1 amd64 [installed]
cuda-toolkit-12-4/unknown,now 12.4.1-1 amd64 [installed]
```


### CUDA Toolkit Installation

Reference the official NVIDIA documentation: https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html

**Installation Commands:**

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt update
apt install cuda-toolkit-12-4
```

**Manual CUDA Installation (Alternative Method):**

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run -O cuda_12.4.0_550.54.14_linux.run
sudo sh cuda_12.4.0_550.54.14_linux.run --toolkit --silent

echo "export PATH=/usr/local/cuda-12.4/bin:\$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc
```


### Verification

After installation and reboot, verify everything is working:

```bash
# Check NVIDIA driver
nvidia-smi

# Check CUDA installation  
nvcc --version
```

Expected nvidia-smi output:

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

**NVLink Verification:**

```bash
nvidia-smi topo -m
```

Expected output:

```
        GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
GPU0     X      NV2     0-3     0               N/A
GPU1    NV2      X      0-3     0               N/A

Legend:
  X    = Self
  NV#  = Connection traversing a bonded set of # NVLinks
```

**CUDA Compiler Verification:**

```bash
nvcc --version
```

Expected output:

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Tue_Feb_27_16:19:38_PST_2024
Cuda compilation tools, release 12.4, V12.4.99
Build cuda_12.4.r12.4/compiler.33961263_0
```


## Ollama Installation and Configuration

Ollama serves as the backbone of our LLM infrastructure, providing a simple API for running various language models locally.

### Installation

Use the official installation script for the most reliable setup:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```


### Service Configuration

Create a dedicated systemd service for Ollama:

**File: `/etc/systemd/system/ollama.service`**

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


### Model Management

Once Ollama is running, you can download and manage models:

```bash
# List available models
ollama list

# Pull specific models
ollama pull deepseek-r1:8b-0528-qwen3-q4_K_M
ollama pull qwen2.5vl:7b
ollama pull snowflake-arctic-embed2:latest
```

**Current Model Inventory:**

```
NAME                                                              ID              SIZE      MODIFIED
deepseek-r1:8b-0528-qwen3-q8_0                                    cade62fd2850    8.9 GB    4 days ago
deepseek-r1:8b-0528-qwen3-q4_K_M                                  6995872bfe4c    5.2 GB    4 days ago
qwen2.5vl:7b                                                      5de93a84837d    6.0 GB    12 days ago
snowflake-arctic-embed2:latest                                    5de93a84837d    1.2 GB    12 days ago
```

**Total VRAM Usage on GPU0: 21.3GB**

Source: https://github.com/ollama/ollama

## Python 3.11 Installation

This AI system requires Python 3.11.13, which offers optimal performance and compatibility with all required packages.

### Why Python 3.11.13?

Based on extensive commercial and government project experience, Python 3.11.13 represents the most stable and reliable version for AI projects as of July 2025. It provides:

- Excellent performance optimizations
- Strong compatibility with AI/ML libraries
- Long-term support and security updates
- Proven stability in production environments


### Installation Process

**Source Installation URL**: https://www.python.org/downloads/release/python-31113/
**Download**: https://www.python.org/ftp/python/3.11.13/Python-3.11.13.tgz

**Virtual Environment Creation**: Follow the official Python documentation: https://docs.python.org/3/library/venv.html

## Open WebUI Installation

Open WebUI provides a sophisticated ChatGPT-like interface for interacting with your locally-hosted language models.

### Overview

Open WebUI is a powerful, feature-rich web interface that offers:

- **ChatGPT-like Experience**: Familiar interface for users
- **Multi-Model Support**: Switch between different LLMs seamlessly
- **Advanced Features**: RAG capabilities, function calling, and more
- **Complete Privacy**: All processing happens locally
- **WebSocket Support**: Real-time communication for responsive interactions


### Installation Steps

**Source URL**: https://github.com/open-webui/open-webui

#### Step 1: Create Dedicated User Account

```bash
# Create linux account with sudo privileges
sudo adduser --system --group --home /usr/local/openwebui openwebui
sudo usermod -aG sudo openwebui

# Set password for the account
sudo passwd openwebui
```


#### Step 2: Install Optimized Python

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


#### Step 3: Create Virtual Environment

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


#### Step 4: Install Open WebUI and Dependencies

```bash
# Install core packages with specific versions for stability
pip install "open-webui==0.6.15" "chromadb==0.6.3" "packaging==23.2" "numpy==1.26.4" "googleapis-common-protos==1.63.2" "uvicorn==0.34.2" "pydantic==2.10.6" "onnxruntime==1.20.1"
```


### Configuration

Open WebUI runs on port 8080 with WebSocket support for real-time features.

**Important Configuration Note**: Set `DATA_DIR=/var/OpenWebUI/data` to separate user data from the application. This enables:

- Easy daily backups
- Clean application updates
- Better data management


### Nginx Configuration

The nginx configuration handles both regular HTTP requests and WebSocket connections:

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
   proxy_set_header Host $host;  # Preserve the original host header
   proxy_set_header X-Real-IP $remote_addr;  # Forward real IP
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # Forward forwarded-for IP
   proxy_set_header X-Forwarded-Proto $scheme;  # Forward protocol (http or https)
   proxy_cache_bypass $http_upgrade;  # Bypass cache for WebSocket connections

   # WebSocket specific timeouts
   proxy_read_timeout 86400;
   proxy_send_timeout 86400;
}

# Main application configuration
location / {  # Handle other requests
    proxy_pass http://openwebui_backend;
    proxy_http_version      1.1;
    # WebSocket support for real-time features
    proxy_set_header        Upgrade $http_upgrade;
    proxy_set_header        Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```


## SearXNG Installation - Private Metasearch Engine

SearXNG is a free, open-source internet metasearch engine that aggregates results from multiple search engines while preserving your privacy.

### Why SearXNG?

SearXNG provides crucial capabilities for AI systems:

- **Privacy Protection**: No tracking, no data collection
- **RAG Integration**: JSON API for Retrieval-Augmented Generation
- **Multiple Sources**: Aggregates results from various search engines
- **Customizable**: Prioritize specific domains and sources
- **Tor Support**: Enhanced privacy through onion routing


### Installation Process

**Source Installation URL**: https://github.com/searxng/searxng

#### Step 1: Create Dedicated User Account

```bash
# Create linux account with sudo privileges
sudo adduser --system --group --home /usr/local/searxng searxng
sudo usermod -aG sudo searxng

# Set password for the account
sudo passwd searxng
```


#### Step 2: Install Optimized Python

```bash
# Use the same Python source from /tmp
cd /tmp/Python-3.11.13

# Clean previous build
make clean

# Configure with optimizations for SearXNG
./configure --prefix=/usr/local/searxng/python --enable-optimizations --with-system-ffi --with-computed-gotos --enable-loadable-sqlite-extensions

# Compile with parallel processing
make -j4
make install

# Add Python path to profile
echo 'export PATH="/usr/local/searxng/python/bin:$PATH"' >> /usr/local/searxng/.profile
```


#### Step 3: Create Virtual Environment and Install Dependencies

```bash
# Switch to searxng user
sudo -u searxng bash

# Create virtual environment
/usr/local/searxng/python/bin/python3.11 -m venv searx-pyenv

# Activate environment
source ~/searx-pyenv/bin/activate

# Add activation to profile
echo "source ~/searx-pyenv/bin/activate" >> ~/.profile

# Upgrade pip and install base packages
pip install --upgrade pip setuptools wheel
pip install uwsgi PyYAML

# Clone SearXNG source
git clone https://github.com/searxng/searxng.git searxng-src
cd searxng-src

# Install SearXNG and dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt
pip install -e . --no-build-isolation
```


### Configuration Files

#### uWSGI Configuration

**File: `/usr/local/searxng/uwsgi.ini`**

```ini
[uwsgi]
# User and group
uid = searxng
gid = searxng

# SearXNG application
module = searx.webapp
callable = app
pythonpath = /usr/local/searxng/searxng-src

# Plugin and interpreter config
single-interpreter = true
master = true
plugin = /usr/lib/uwsgi/plugins/python3_plugin.so
lazy-apps = true
enable-threads = true

# Worker configuration
workers = %k
threads = 2

# Socket configuration
socket = /usr/local/searxng/run/socket
chmod-socket = 666

# Environment variables
env = SEARXNG_SETTINGS_PATH=/etc/searxng/settings.yml

# Logging
disable-logging = true

# Performance tuning
cheaper-algo = busyness
processes = 4
cheaper = 1
cheaper-initial = 1
cheaper-overload = 5
cheaper-step = 1
cheaper-busyness-multiplier = 20
cheaper-busyness-min = 10
cheaper-busyness-max = 70
cheaper-busyness-backlog-alert = 16
cheaper-busyness-backlog-step = 2

# Process management
idle = 0
die-on-idle = false
max-requests = 1000
max-worker-lifetime = 86400
reload-on-rss = 128

# Buffer settings
buffer-size = 32768
post-buffering = 8192
post-buffering-bufsize = 65536
```


#### SearXNG Settings

**File: `/etc/searxng/settings.yml`**

```yaml
use_default_settings: true

general:
  debug: false

brand:
  instance_name: "AI. RAG Search for Demo"

hostnames:
  high_priority:
    # Government domains (specific)
    - '(.*\.)?gov\.hk$'
    - '(.*\.)?info\.gov\.hk$'  # Existing gov.hk subdomain

    # Government domains (wildcard - matches any gov.<tld>)
    - '(.*\.)?gov\.[a-z]{2,}$'  # Prioritizes all government TLDs (e.g., .gov.au, .gov.br)

    # Major news domains (extracted from URLs)
    - '(.*\.)?hongkongfp\.com$'
    - '(.*\.)?scmp\.com$'
    - '(.*\.)?rthk\.hk$'
    - '(.*\.)?hk01\.com$'
    - '(.*\.)?hkexnews\.hk$'
    - '(.*\.)?thestandard\.com\.hk$'
    - '(.*\.)?cnn\.com$'
    - '(.*\.)?news\.cn$'        # Matches english.news.cn, news.cn, etc.
    - '(.*\.)?bloomberg\.com$'

    # TLD priorities (catch-all for these extensions)
    - '(.*\.)?[^.]*\.hk$'      # Any .hk domain
    - '(.*\.)?[^.]*\.com$'     # Any .com domain
    - '(.*\.)?[^.]*\.ai$'      # Any .ai domain

categories_as_tabs:
  general: {}
  news: {}
  it: {}
  science: {}
  onions: {}

search:
  safe_search: 0
  autocomplete: "google"
  max_results: 10
  formats:
    - html
    - json
  default_params:
    - "after:2025-01-01"
    - "language:auto"
    - "safesearch:0"

server:
  secret_key: "$(openssl rand -base64 64)"
  limiter: true
  image_proxy: false
  default_http_headers:
    Referrer-Policy: no-referrer
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block

valkey:
  url: valkey://:<password>@127.0.0.1:6379/0

ui:
  static_use_hash: true
  results_on_new_tab: false

engines:
  - name: google
    engine: google
    shortcut: g
    timeout: 10.0
    categories: [general, news, it, science]
    weight: 2.0
    proxy: tor
    tokens: []
    custom_params:
      nfpr: "1"
      filter: "0"
      safe: "off"

# Onion engines for dark web research
  - name: ahmia
    engine: ahmia
    shortcut: ah
    categories: [onions]
    timeout: 20.0
    proxy: tor
    disabled: false

# Disable other engines for minimal setup
disabled_engines:
  - bing
  - yahoo
  - yandex
  - startpage
  - qwant

# Optimized outgoing configuration for Tor
outgoing:
  verify_tls: true
  use_tor_proxy: true
  tor_proxy: socks5h://127.0.0.1:9050
  max_request_timeout: 120.0
  max_connections: 100
  retries: 3
  pool_connections: 50
  pool_maxsize: 20
  request_headers:
    User-Agent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
    Accept: 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8'
    Accept-Language: 'en-US,en;q=0.5'
    Accept-Encoding: 'gzip, deflate'
    Connection: 'keep-alive'
    Upgrade-Insecure-Requests: '1'

plugins:
  searx.plugins.tor_check.SXNGPlugin:
    active: false
  searx.plugins.self_info.SXNGPlugin:
    active: true
  searx.plugins.tracker_url_remover.SXNGPlugin:
    active: true

query_processing:
  auto_inject_params:
    - "-filetype:pdf"
    - "-filetype:ppt"
    - "-filetype:doc"
  site_boost:
    - ".gov.hk"
    - ".edu.hk"
    - ".org.hk"
    - ".com.hk"
  language_detection: true
  query_enhancement:
    enable_synonyms: true
    auto_correct: true

performance:
  cache:
    enabled: true
    ttl: 3600
    valkey_key_prefix: "searxng_cache:"
  rate_limit:
    enabled: true
    requests_per_minute: 60
    burst_limit: 10
  connection_pool:
    max_connections: 100
    max_keepalive: 20
    keepalive_timeout: 30
```


#### SystemD Service Configuration

**File: `/etc/systemd/system/searxng.service`**

```ini
[Unit]
Description=SearXNG uWSGI Service
After=network.target

[Service]
Type=notify
User=searxng
Group=searxng
WorkingDirectory=/usr/local/searxng/searxng-src
Environment=PATH=/usr/local/searxng/searx-pyenv/bin
Environment=SEARXNG_SETTINGS_PATH=/etc/searxng/settings.yml
ExecStart=/usr/local/searxng/searx-pyenv/bin/uwsgi --ini /usr/local/searxng/uwsgi.ini
KillSignal=SIGQUIT
Restart=on-failure
RestartSec=5
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```


### Nginx Configuration for SearXNG

```nginx
upstream searxng_backend {
  server unix:/usr/local/searxng/run/socket;
  keepalive 32;
  keepalive_requests 100;
  keepalive_timeout 60s;
}

# Direct web search endpoint for Open-WebUI
location /search {
  include uwsgi_params;
  uwsgi_pass searxng_backend;

  # Critical parameters for proper functionality
  uwsgi_param HTTP_X_REAL_IP $remote_addr;
  uwsgi_param HTTP_X_FORWARDED_FOR $proxy_add_x_forwarded_for;
  uwsgi_param HTTP_X_FORWARDED_PROTO $scheme;
  uwsgi_param HTTP_X_FORWARDED_HOST $host;  

  # Buffer optimization
  uwsgi_buffering on;
  uwsgi_buffer_size 128k;
  uwsgi_buffers 16 128k;
  uwsgi_busy_buffers_size 256k;
  uwsgi_max_temp_file_size 0;
  uwsgi_temp_file_write_size 256k;   

  # Timeouts
  uwsgi_connect_timeout 60s;
  uwsgi_send_timeout 300s;
  uwsgi_read_timeout 300s;

  # Enable CORS for API access
  add_header 'Access-Control-Allow-Origin' '*';
  add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
  add_header 'Access-Control-Allow-Headers' 'Content-Type';
  }

location /searxng/static {
  alias /usr/local/searxng/searxng-src/searx/static;
  expires 1y;
  access_log off;
  gzip_static on;
  add_header Cache-Control "public, immutable";
}   
```


### Service Status Verification

After installation, verify the service is running properly:

```bash
systemctl status searxng.service
```

Expected output:

```
● searxng.service - SearXNG uWSGI Service
     Loaded: loaded (/etc/systemd/system/searxng.service; enabled; preset: enabled)
     Active: active (running) since Sat 2025-07-12 16:28:00 HKT; 6s ago
   Main PID: 21275 (uwsgi)
     Status: "uWSGI is ready"
      Tasks: 9 (limit: 18919)
     Memory: 101.6M (peak: 108.2M)
        CPU: 1.199s
     CGroup: /system.slice/searxng.service
             ├─21275 /usr/local/searxng/searx-pyenv/bin/uwsgi --ini /usr/local/searxng/uwsgi.ini
             └─21276 /usr/local/searxng/searx-pyenv/bin/uwsgi --ini /usr/local/searxng/uwsgi.ini
```

**Note**: Valkey server is required for SearXNG caching functionality.

**Reference**: https://docs.searxng.org/admin/installation-searxng.html

## ComfyUI Installation - Advanced Image Generation

ComfyUI is a powerful, node-based interface for Stable Diffusion that offers unparalleled control over image generation workflows.

### Why ComfyUI?

ComfyUI provides professional-grade image generation capabilities:

- **Node-based Workflow**: Visual programming for complex pipelines
- **Advanced Control**: Precise control over every aspect of generation
- **Model Flexibility**: Support for various Stable Diffusion models
- **Custom Nodes**: Extensible architecture for specialized functions
- **Batch Processing**: Efficient handling of multiple images


### Installation Process

**Source Installation URL**: https://github.com/comfyanonymous/ComfyUI

#### Step 1: Create Dedicated User Account

```bash
# Create linux account with sudo privileges
sudo adduser --system --group --home /usr/local/comfyui comfyui
sudo usermod -aG sudo comfyui

# Set password for the account
sudo passwd comfyui
```


#### Step 2: Install Optimized Python

```bash
# Use the same Python source from /tmp
cd /tmp/Python-3.11.13

# Clean previous build
make clean

# Configure with optimizations for ComfyUI
./configure --prefix=/usr/local/comfyui/python --enable-optimizations --enable-shared --with-system-ffi LDFLAGS="-Wl,-rpath=/usr/local/comfyui/lib"

# Compile with parallel processing
make -j4
make install

# Add Python path to profile
echo 'export PATH="/usr/local/comfyui/python/bin:$PATH"' >> /usr/local/comfyui/.profile
```


#### Step 3: Create Virtual Environment and Install Dependencies

```bash
# Switch to comfyui user
sudo -u comfyui bash

# Create virtual environment
python -m venv $HOME/comfy

# Activate environment
source ~/comfy/bin/activate

# Add activation to profile
echo "source ~/comfy/bin/activate" >> ~/.profile

# Install PyTorch with CUDA support
pip install xformers --index-url https://download.pytorch.org/whl/cu124
pip install torch==2.6.0+cu124 torchvision torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cu124

# Clone ComfyUI
git clone https://github.com/comfyanonymous/ComfyUI.git /usr/local/comfyui/ComfyUI

# Set GPU visibility for dual-GPU setup
export CUDA_VISIBLE_DEVICES=0,1
```


### Startup Script

Create a convenient startup script for ComfyUI:

**File: `/usr/local/comfyui/start.sh`**

```bash
#!/bin/bash
cd /usr/local/comfyui
source ../comfy/bin/activate

# Multi-GPU optimization for dual RTX 2080Ti
export CUDA_VISIBLE_DEVICES=1,0

python main.py --listen 0.0.0.0 --port 8188 --enable-cors-header --use-split-cross-attention --fast --fp16-unet --fp16-vae --cuda-malloc
```


### Command Line Parameters Explained

- `--listen 0.0.0.0`: Makes the server accessible from any device on your network
- `--port 8188`: Sets the port for web interface access
- `--enable-cors-header`: Enables cross-origin resource sharing
- `--use-split-cross-attention`: Optimizes memory usage for older GPUs
- `--fast`: Enables fast sampling methods
- `--fp16-unet`: Uses 16-bit precision for U-Net to save VRAM
- `--fp16-vae`: Uses 16-bit precision for VAE to save VRAM
- `--cuda-malloc`: Uses CUDA memory allocation for better performance


### Model Configuration

**Recommended Models for Stable Diffusion 3.5:**

- `sd3.5_large_fp8_scaled.safetensors` (main model)
- `clip_l.safetensors` (text encoder)
- `clip_g.safetensors` (text encoder)
- `t5xxl_fp8_e4m3fn_scaled.safetensors` (text encoder)

**Total VRAM Usage on GPU1**: 20.5GB when generating 768×768 images

### Access Information

After starting ComfyUI, access the interface at: `http://your-server-ip:8188`

## System Access and Credentials

### Public Access

The complete AI system is accessible at: **https://canyouai.com**

**Demo Credentials:**

- **Username**: demo@canyouai.com
- **Password**: canyouai


### Security Considerations

In production environments, ensure:

- Strong passwords for all accounts
- Regular security updates
- Firewall configuration
- SSL/TLS encryption
- Regular backups


## Troubleshooting Guide

### Common Issues and Solutions

#### GPU Driver Issues

**Problem**: `nvidia-smi` command not found
**Solution**:

```bash
# Check if drivers are installed
dpkg -l | grep nvidia

# Reinstall if necessary
sudo ubuntu-drivers install
sudo reboot
```

**Problem**: CUDA out of memory errors
**Solution**:

```bash
# Check GPU memory usage
nvidia-smi

# Reduce model precision or batch size
# For Ollama: Use smaller quantized models
# For ComfyUI: Enable fp16 options
```


#### Service Issues

**Problem**: Service fails to start
**Solution**:

```bash
# Check service status and logs
systemctl status service-name
journalctl -u service-name -f

# Check file permissions
ls -la /usr/local/service-directory/
```

**Problem**: Port conflicts
**Solution**:

```bash
# Check what's using the port (Ubuntu 24.04 LTS replacement for netstat)
sudo ss -tulpn | grep :8080

# Alternative using lsof
sudo lsof -i :8080

# Kill conflicting processes
sudo kill -9 <process-id>

# Or use fuser to kill processes using specific port
sudo fuser -k 8080/tcp
```


#### Python Environment Issues

**Problem**: Package installation fails
**Solution**:

```bash
# Update pip
pip install --upgrade pip

# Clear pip cache
pip cache purge

# Use specific index for PyTorch
pip install torch --index-url https://download.pytorch.org/whl/cu124
```


### System Monitoring

#### GPU Monitoring

```bash
# Real-time GPU monitoring
watch -n 1 nvidia-smi

# GPU utilization history
nvidia-smi -l 1

# Detailed GPU information
nvidia-smi -q
```


#### Service Monitoring

```bash
# Check all AI services
systemctl status ollama openwebui searxng comfyui

# Monitor system resources
htop
iotop
```


### Port Troubleshooting Commands for Ubuntu 24.04 LTS

Since Ubuntu 24.04 LTS no longer includes `netstat` by default, use these modern alternatives[^1][^2][^3]:

#### Using `ss` Command (Modern netstat replacement)

```bash
# List all listening ports with process information
sudo ss -tulpn

# Check specific port
sudo ss -tulpn | grep :8080

# Show only TCP listening ports
sudo ss -tln

# Show only UDP listening ports  
sudo ss -uln

# Show detailed socket information for specific port
sudo ss --listening --numeric --processes sport :8080
```


#### Using `lsof` Command

```bash
# Show all processes using network connections
sudo lsof -i -P -n | grep LISTEN

# Check specific port
sudo lsof -i :8080

# Show processes using TCP port
sudo lsof -i TCP:8080

# Show processes using UDP port
sudo lsof -i UDP:8080
```


#### Advanced Port Troubleshooting

```bash
# Kill process using specific port with lsof
sudo kill -9 $(sudo lsof -t -i:8080)

# Kill process using fuser
sudo fuser -k 8080/tcp

# Check if port is reachable
nc -zv localhost 8080

# Scan port range
nmap -p 8080-8188 localhost
```


## Performance Optimization

### Memory Management

**Swap Configuration:**

```bash
# Check current swap
swapon --show

# Optimize swappiness for AI workloads
echo 'vm.swappiness=10' >> /etc/sysctl.conf
```

**VRAM Optimization:**

- Use quantized models when possible
- Enable fp16 precision for image generation
- Monitor VRAM usage and adjust batch sizes


### Storage Optimization

**SSD Performance:**

```bash
# Enable TRIM for SSD longevity
sudo systemctl enable fstrim.timer

# Check SSD health
sudo smartctl -a /dev/sda
```

**Directory Organization:**

- Keep models in `/usr/local/*/models/`
- Store user data in `/var/*/data/`
- Regular cleanup of temporary files


## Future Enhancements

### Part 2: Advanced RAG and Pipeline Configuration

Coming soon - Part 2 will cover:

- **Advanced RAG Pipelines**: ChromaDB integration with Open WebUI
- **Custom Functions**: Building specialized AI functions
- **Multi-Model Workflows**: Orchestrating different models
- **Performance Tuning**: Optimizing for specific use cases


### Part 3: Advanced Image Generation Workflows

Coming soon - Part 3 will cover:

- **Stable Diffusion 3.5 Workflows**: Advanced ComfyUI setups
- **ControlNet Integration**: Precise image control
- **Custom Node Development**: Building specialized nodes
- **Batch Processing**: Automated image generation pipelines


## Conclusion

This tutorial has guided you through building a comprehensive, self-hosted AI system that rivals commercial offerings while maintaining complete privacy and control. The system you've built includes:

- **Local LLM serving** with Ollama
- **Professional chat interface** with Open WebUI
- **Private search capabilities** with SearXNG
- **Advanced image generation** with ComfyUI
- **Production-ready deployment** with Nginx


### Key Achievements

1. **Privacy Protection**: All AI processing happens locally
2. **Cost Efficiency**: No recurring subscription fees
3. **Customization**: Full control over models and configurations
4. **Scalability**: Modular architecture allows easy expansion
5. **Security**: Dedicated user accounts and service isolation

### Next Steps

With your foundation in place, you can:

- Experiment with different LLM models
- Develop custom workflows in ComfyUI
- Integrate external data sources for RAG
- Build custom applications using the APIs
- Scale the system with additional hardware

Remember: You don't need an MBA or PhD to work with AI projects - you just need Linux skills, Python knowledge, and the determination to build something amazing.

**Total System Resources:**

- **CPU**: 4 cores efficiently utilized
- **Memory**: 16GB optimally allocated
- **Storage**: 4TB with proper partitioning
- **GPU0**: 21.3GB VRAM for LLMs
- **GPU1**: 20.5GB VRAM for image generation

This represents a complete, production-ready AI system built with open-source tools and modest hardware investments. Enjoy your new local AI capabilities!
