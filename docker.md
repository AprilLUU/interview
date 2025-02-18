## 安装

获取软件对应key

```shell
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

添加软件资源

```shell
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新apt

```shell
 sudo apt update
```

安装docker

```shell
 sudo apt install docker-ce
```

## 配置docker代理和镜像

创建daemon配置文件

```shell
sudo nano /etc/docker/daemon.json
```

添加如下配置，注意https-proxy需配置为http://127.0.0.1:7897，配置为https://127.0.0.1:7897会连接出错

```json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:7897",
    "https-proxy": "http://127.0.0.1:7897",
    "no-proxy": "127.0.0.1"
  },
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://ccr.ccs.tencentyun.com"
  ]
}
```

## 构建镜像

Dockfile

```dockerfile
# 基础镜像，使用 CUDA Runtime
FROM nvidia/cuda:12.3.0-runtime-ubuntu22.04

# 设置环境变量
# 防止在 apt-get install 时出现交互式提示，以便命令可以无阻碍运行
ENV DEBIAN_FRONTEND=noninteractive
# 让 Python 的输出立即显示（默认会缓冲），方便调试和日志记录
ENV PYTHONUNBUFFERED=1
# 设置容器的语言环境为 UTF-8
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# 安装必要工具
# 22.04默认只提供python3.10以上
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.10 python3-pip python3.10-dev build-essential && \
    rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /app

# 安装 Python 依赖
RUN pip3 install --no-cache-dir deepface

# 拷贝项目文件
COPY ./script/face_recognition.py /app/

# 拷贝模型权重
COPY ./script/model/ /root/.deepface/weights/

# 设置默认运行命令
CMD ["python3", "face_recognition.py"]

```

拉取镜像出错，可以提前拉取镜像

```shell
sudo docker pull nvidia/cuda:12.3.0-runtime-ubuntu22.04
```

