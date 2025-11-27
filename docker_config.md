# 🚀 Ubuntu 22.04 纯净环境配置 verl 全流程指南

**环境假设：**
*   系统：Ubuntu 22.04 LTS (纯净版)
*   显卡：NVIDIA GPU (如 RTX 2080 Ti / 3090 / 4090 等)
*   目标：使用 Docker 部署 verl 大模型训练框架
*   网络：国内网络环境（需要配置镜像加速）

---

## 第一阶段：基础工具与 Docker 安装

由于是纯净系统，首先需要安装基础下载工具，然后安装 Docker。

### 1. 更新软件源并安装 curl
```bash
sudo apt update
sudo apt install -y curl git
```

### 2. 安装 Docker
使用官方的一键安装脚本（这是最快的方法）：
```bash
curl -fsSL https://get.docker.com | bash
```

### 3. 启动 Docker 并设置开机自启
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### 4. 测试 Docker 是否存活
拉取一个测试镜像，验证 Docker 守护进程是否正常：
```bash
sudo docker run hello-world
```
*   **预期结果**：输出 "Hello from Docker!" 即为成功。
*   **常见问题**：如果此时拉取卡住或报错 `Timeout`，不要急，我们在第三阶段会解决网络问题。只要命令能执行不报 `Command not found` 即可。

---

## 第二阶段：显卡驱动安装与升级

为了支持最新的 CUDA 版本（verl 通常需要 CUDA 12.1+），我们需要安装较新的显卡驱动。

### 1. 检查推荐驱动
查看系统推荐的驱动版本：
```bash
ubuntu-drivers devices
```
*   **操作**：在输出列表中找到带有 `recommended` 字样的版本，或者数字最大的版本（例如 `nvidia-driver-580` 或 `550`）。

### 2. 安装高性能驱动
不要安装旧版（如 535），直接一步到位安装 550 或 580 以上版本，以获得 CUDA 12.x 的完整支持。
*(将下面的 580 替换为你查到的推荐版本号)*
```bash
# 停止 Docker 防止冲突
sudo systemctl stop docker

# 安装驱动
sudo apt install -y nvidia-driver-580
```

### 3. 重启服务器（必须）
驱动涉及内核模块，必须重启生效：
```bash
reboot
```

### 4. 验证驱动
重启后重新连接服务器，输入：
```bash
nvidia-smi
```
*   **检查点**：
    *   Driver Version 应 >= 550.x
    *   CUDA Version 应 >= 12.4
    *   显卡型号正确显示。

---

## 第三阶段：解决网络问题与配置 NVIDIA 运行时

这是最关键的一步，解决“下载慢/超时”以及“Docker 无法调用显卡”的问题。

### 1. 安装 NVIDIA Container Toolkit
让 Docker 能够识别 GPU：
```bash
# 添加源
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# 安装工具包
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### 2. 配置 Docker 镜像加速与 GPU 运行时（合二为一）
我们直接手动编辑配置文件，同时解决**下载慢**和**GPU支持**两个问题。

编辑文件：
```bash
sudo vim /etc/docker/daemon.json
```
*(按 `i` 进入编辑模式，粘贴以下内容)*

```json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live",
        "https://hub.rat.dev"
    ],
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
*(按 `Esc`，输入 `:wq` 保存退出)*

### 3. 重启 Docker 服务
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 4. 验证配置
测试 Docker 是否能调用显卡：
```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
*   **预期结果**：打印出显卡信息表格。如果成功，说明环境已经完美。

---

## 第四阶段：部署 verl 环境

### 1. 下载 verl 源码
```bash
cd ~
git clone https://github.com/volcengine/verl.git
cd verl
```

### 2. 拉取 Docker 镜像
得益于刚才配置的加速源，这一步应该会比较快：
```bash
docker pull verlai/verl:vllm011.latest
```

### 3. 启动容器（解决 Entrypoint 闪退问题）
**注意**：官方镜像默认会启动 vLLM 服务，导致传参 `sleep infinity` 时报错闪退。我们需要修改入口点为 `/bin/bash`。

请在 `~/verl` 目录下直接执行：

```bash
docker run -d --name verl \
    --runtime=nvidia \
    --gpus all \
    --net=host \
    --shm-size="10g" \
    --cap-add=SYS_ADMIN \
    -v $(pwd):/workspace/verl \
    --entrypoint /bin/bash \
    verlai/verl:vllm011.latest \
    -c "sleep infinity"
```

### 4. 检查容器状态
```bash
docker ps
```
确保容器 `verl` 的状态是 `Up`。

---

## 第五阶段：容器内安装与验证

### 1. 进入容器
```bash
docker exec -it verl bash
```

### 2. 切换目录并安装
进入容器后，提示符会改变。执行以下命令：
```bash
# 切换到挂载的源码目录
cd /workspace/verl

# 以编辑模式安装 verl
pip3 install --no-deps -e .
```

### 3. 最终验证
安装完成后，运行 Python 脚本测试环境：
```bash
python3 -c "import torch; import verl; print(f'\n[SUCCESS] GPU: {torch.cuda.get_device_name(0)}'); print(f'[SUCCESS] Verl Version: {verl.__version__}')"
```

**如果看到 `[SUCCESS]` 字样和显卡型号，恭喜你！所有配置工作圆满完成！** 🎉