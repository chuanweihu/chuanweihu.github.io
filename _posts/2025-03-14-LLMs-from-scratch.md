---
layout: post
title:  谈谈从零开始搭建大模型
categories: [blog]
comments: true
tags: [Rocky, LLM]
---

> 自从在网上使用了DeepSeek-R1之后，发现写代码的工作可以变得非常简单。恰好公司的服务器最近使用率低，可以从内网匀出一台来搭建大模型。公司的服务器没有用于显示用的GPU卡，导致开始的图形化界面时遇到了一些问题，不过也解决了。搭建模型的过程中趟了一些坑，不过感觉费时间的是熟悉工具的使用，没什么难度。将模型搭建起来后，感觉写代码的工作省事不少。

* TOC
{:toc}

<!--more-->

## 搭建Rocky系统

> 之前公司一直用的是CentOS7集群服务器，内网又不好调，只好将就着用。现在空闲下来，可以用来装最新的系统，想来想去还是安装CentOS东家的Rocky9系统吧。
> Rocky9系统的安装和其他系统的安装没什么区别，有一个有意思的地方是图形界面的安装是类似PE形式的安装方式。
> 首先，下载Rocky系统和校验文件，通过`sha256sum`命令验证文件的完整性。
> 其次，制作启动U盘。以前一直用`UltraISO`制作，同事推荐使用`Rufus`，确实短小精悍。要是有Linux系统的话，直接用`dd`命令写入即可。
> 最后，安装系统需要先在`BIOS`中调整启动顺序，将U盘调整到首位即可，安装过程简单，分区可以先自动分区，然后根据需要调整。

通过计算SHA256校验值验证文件完整性：

`$ sha256sum openEuler-22.03-LTS-SP2-aarch64-dvd.iso`

## 安装GPU驱动

> 由于公司的服务器没有用于显示的GPU卡，只有3张NVIDIA V100计算卡，所以启用默认的Nouveau驱动，并且安装NVIDIA驱动后需要将NVIDIA驱动不做为主显示设备。

### 安装必要工具及依赖

1. 启用EPEL(Extra Packages for Enterprise Linux)仓库：`sudo dnf install epel-release -y`（获取额外依赖包）
2. 安装系统开发工具组：`sudo dnf groupinstall "Development Tools" -y` 
3. 安装内核开发包：`sudo dnf install kernel-devel-$(uname -r)`（确保与当前内核版本一致）
4. 安装动态内核模块支持工具DKMS（Dynamic Kernel Module Support）：`sudo dnf install dkms -y`

### 安装NVIDIA驱动

1. 添加CUDA仓库（根据系统版本选择路径）：`sudo dnf config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/rhel9/$(uname -i)/cuda-rhel9.repo`
2. 安装编译工具链：`sudo dnf install kernel-headers-$(uname -r) kernel-devel-$(uname -r) tar bzip2 make automake gcc gcc-c++ pciutils elfutils-libelf-devel libglvnd-opengl libglvnd-glx libglvnd-devel acpid pkgconf dkms -y`
3. 通过DKMS安装最新驱动：`sudo dnf module install nvidia-driver:latest-dkms -y`

### 禁用Nouveau驱动（用GPU卡为主显示设备时）

```shell
# 修改Xorg显示服务器配置
[admin@localhost /]$ vi /etc/X11/xorg.conf.d/10-nvidia.conf
Section "OutputClass" # 定义输出类规则，用于匹配特定显卡驱动
    Identifier "nvidia" # 配置块唯一标识符（可自定义）
    MatchDriver "nvidia-drm" # 仅当系统使用 NVIDIA 的 DRM（Direct Rendering Manager）驱动时生效
    Driver "nvidia" # 指定使用 NVIDIA 专有驱动（而非开源 nouveau 驱动）
    Option "AllowEmptyInitialConfiguration" # 允许 X Server 在无连接显示器的情况下启动
   Option "PrimaryGPU" "no" # 在多 GPU 系统中不将此显卡设为主显示设备
   Option "SLI" "Off" # 禁用 SLI 多卡交火模式（适用于多 NVIDIA 显卡但无需协同渲染的情况）
   Option "BaseMosaic" "off" # 关闭基础马赛克模式（多显示器拼接显示）
EndSection

# 修改GRUB引导参数
[admin@localhost /]$ sudo grubby --args="nouveau.modeset=0 rd.driver.blacklist=nouveau" --update-kernel=ALL

# 重启
[admin@localhost /]$ sudo reboot now
```

## 部署LLMs

> 考虑到服务器的资源需要共享使用，而且使用人员的层次参差不齐。笔者使用了CherryStdio作为普通用户的界面，LobeChat作为开发人员的界面，并通过Higress开源API网关来调整访问资源。
>
> 其次，Docker容器比较好的将服务器的资源分割开来，便于以后搭建集群。所以将3张GPU划分出3个容器应用，然后再通过API访问相关的大模型。
>
> 最后，为了只下载一次大模型，通过Ollama将模型文件下载到本地，然后通过创建容器Ollama应用进行挂载使用。

```Mermaid
graph TD
    %% 定义节点关系
    用户[用户/开发人员]:::user --> |输入对话| LobeChat/Cherry_Stdio:::lobe
    LobeChat/Cherry_Stdio --> |API请求| API网关:::api
    API网关 --> |分发| ollama1:::ollama
    API网关 --> |分发| ollama2:::ollama
    API网关 --> |分发| ollama3:::ollama
    ollama1 --> GPU1:::gpu
    ollama2 --> GPU2:::gpu
    ollama3 --> GPU3:::gpu
```

### 主机本地部署Ollama

> Ollama 是一个专为本地机上便捷部署和运行大型语言模型（LLM）而设计的开源框架，它可以用简单的命令行部署多种大模型。Ollama的部署很简单，直接下载相应的安装脚本，然后启动Ollama服务就可以了。由于国内下载Github的速度较慢，可以使用国内镜像代理加速Github文件的下载。

#### 下载安装ollama脚本

```shell
# 下载安装脚本
[admin@localhost ~]$ curl -fsSL https://ollama.com/install.sh -o ollama_install.sh
# 将地址替换为镜像代理
[admin@localhost ~]$ sed -i 's|https://ollama.com/download/ollama-linux|https://ghproxy.cn/https://github.com/ollama/ollama/releases/download/v0.5.11/ollama-linux|g' ollama_install.sh
```

#### 运行安装脚本

```shell
# 加入执行权限
[admin@localhost ~]$ chmod +x ollama_install.sh
# 运行安装脚本
[admin@localhost ~]$ sh ollama_install.sh
```

#### 启动ollama服务

```shell
# 重新加载 systemd 服务配置文件
[admin@localhost ~]$ sudo systemctl daemon-reload
# 立即启动名为 ollama 的 systemd 服务
[admin@localhost ~]$ sudo systemctl start ollama
# 设置 Ollama 服务开机自动启动
[admin@localhost ~]$ sudo systemctl enable ollama
```

#### 下载并运行大模型

```shell
# 下载Deepseek模型
[admin@localhost ~]$ ollama pull deepseek-r1:1.5b    # 测试
# 启动Deepseek模型并进入交互式对话模式
[admin@localhost ~]$ ollama run deepseek-r1:1.5b    # 测试
[admin@localhost ~]$ ollama run deepseek-r1:32b
[admin@localhost ~]$ ollama run deepseek-coder:33b 
```

#### ollama的常用命令

> 模型管理（pull、list、rm、cp、create、push）
> 服务控制（serve、start、stop）
> 信息查看（show、ps、version、help）
> 环境配置（OLLAMA_HOST、OLLAMA_MODELS、CUDA_VISIBLE_DEVICES）

```shell
# 列出本地已下载的所有大语言模型
[admin@localhost ~]$ ollama list
# 查看当前运行的模型实例
[admin@localhost ~]$ ollama ps -a
# 显示模型详细信息
[admin@localhost ~]$ ollama show deepseek-r1:1.5b
# 显示模型文件信息并保存
[admin@localhost ~]$ ollama show --modefile deepseek-r1:1.5b Modelfile
# 基于模型文件信息创建模型
[admin@localhost ~]$ ollama create deepseek-r1-new:1.5b -f ./Modelfile
```

#### 测试API访问Ollama模型

```shell
# `http://localhost:11434`: 模型服务地址
# `/v1/chat/completions`: OpenAI 兼容的对话补全接口路径
# `-H "Content-Type: application/json"`: 声明请求体为 JSON 格式，确保服务端正确解析数据。
# -d 参数中的 JSON 数据：指定调用的模型名称和对话历史数组
curl http://localhost:11434/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "deepseek-r1:1.5b",  
    "messages": [    
        {"role": "user", "content": "hello"}  
    ]
}'
```

### 主机部署容器Ollama

> 由于有3个GPU计算卡，所有准备使用3个Ollama容器来使用GPU。由于Ollama默认调用端口是11434，可以将每一个Ollama容器应用端口（11434）映射为宿主机不同的端口即可使用多个Ollama容器。
> 这里的使用的GPU是NVIDIA的Tesla V100，需要安装Nvidia的容器工具包才能使用。由于国内无法访问 DockerHub，使用代理地址访问。

#### docker下载ollama镜像

```shell
# 代理地址下载ollama
docker pull docker.lms.run/ollama/ollama:0.5.11
```

#### 安装nvidia容器工具包

`sudo dnf install -y nvidia-container-toolkit`

#### 配置Docker运行时

`sudo nvidia-ctk runtime configure --runtime=docker`

#### 重启Docker服务

```shell
# 重新加载 systemd 服务配置文件
[admin@localhost ~]$ sudo systemctl daemon-reload
# 立即启动名为 docker 的 systemd 服务
[admin@localhost ~]$ sudo systemctl restart docker
# 设置 docker 服务开机自动启动
[admin@localhost ~]$ sudo systemctl enable docker
```

#### 创建并运行ollama容器

```shell
# `docker run`：创建并运行容器
# `-dp 8880:11434`：-d（detached，守护进程模式）和-p（port，端口映射），将宿主机的8880映射到容器的11434
# `--runtime=nvidia`：指定使用NVIDIA的容器运行时，支持GPU访问
# `--gpus device=0`：指定使用设备0的GPU
# `--name DeepSeek-R1-1`：容器名称
# `-v /usr/share/ollama/.ollama/models:/root/.ollama/models`：挂载卷，将宿主机的目录挂载到容器内
# docker.1ms.run/ollama/ollama:0.5.11：使用的镜像

[admin@localhost ~]$ docker run -dp 8880:11434 --runtime=nvidia --gpus device=0 --name DeepSeek-R1-1 -v /usr/share/ollama/.ollama/models:/root/.ollama/models docker.1ms.run/ollama/ollama:0.5.11
[admin@localhost ~]$ docker run -dp 8881:11434 --runtime=nvidia --gpus device=1 --name DeepSeek-R1-2 -v /usr/share/ollama/.ollama/models:/root/.ollama/models docker.1ms.run/ollama/ollama:0.5.11
[admin@localhost ~]$ docker run -dp 8882:11434 --runtime=nvidia --gpus device=2 --name DeepSeek-R1-3 -v /usr/share/ollama/.ollama/models:/root/.ollama/models docker.1ms.run/ollama/ollama:0.5.11
```

#### 进入容器并运行模型

```shell
# 运行docker中ollama
# docker exec -it <你的容器名称或ID> /bin/bash
[admin@localhost ~]$ docker exec -it DeepSeek-R1-1 ollama run deepseek-r1:1.5b
```

#### 查看docker中运行的容器

```shell
# 查看docker中运行的容器
[admin@localhost ~]$ docker ps -a
# 查看是否正常配置GPU
[admin@localhost ~]$ docker info | grep -i nvidia
```

#### 测试API访问容器Ollama模型

```shell
# `curl`：命令行工具，用于发送 HTTP 请求。
# `http://localhost:8880`: 模型服务地址
# `/v1/chat/completions`: OpenAI 兼容的对话补全接口路径
# `-H "Content-Type: application/json"`: 声明请求体为 JSON 格式，确保服务端正确解析数据。
# -d 参数中的 JSON 数据：指定调用的模型名称和对话历史数组
[admin@localhost ~]$ curl http://localhost:8880/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "deepseek-r1:1.5b",  
    "messages": [    
        {"role": "user", "content": "hello"}  
    ]
}'
```

### Higress（云原生API网关）的部署

> Higress 是阿里巴巴开源的云原生 API 网关，基于 Envoy 和 Istio 内核构建，集流量网关、微服务网关、安全网关能力于一体，具备高性能、高扩展性和 AI 原生支持等特性。
> Higress 暴露了三个端口，分别是 HTTP 访问端口 8080，HTTPS 访问端口 8443，以及控制台访问端口 8001。
> 因为 Higress 运行在 Docker 容器中，访问不到服务器内网，所以这里填写的Ollama服务主机名要填服务器的公网地址。
> Higress的安装跟Ollama类似，只要下载安装脚本再运行即可使用，如果需要外网使用，需要删除安装脚本文件中的`127.0.0.1: `。
> Higress的配置需要通过访问控制台来进行，一般地址为`< 公网 ip>:8001`
> 这里需要提到的是调用相应的模型时，model名称为这里自定义的名称，模型的统一调用接口为8080和8443。

#### 下载安装higress脚本

```shell
# # `curl`：命令行工具，用于发送 HTTP 请求。
[admin@localhost ~]$ curl -sS https://higress.cn/ai-gateway/install.sh -o higress_install.sh
```

#### 运行安装higress脚本

```shell
# 内网使用，直接运行脚本
# [admin@localhost ~]$ bash ./higress_install.sh

# 公网使用，删除higress_install.sh文本中的`127.0.0.1: `
[admin@localhost ~]$ vi./higress_install.sh
# 运行脚本
[admin@localhost ~]$ bash ./higress_install.sh
```

#### 进入 Higress 控制台

1. 在浏览器输入`< 公网 ip>:8001`进入 Higress 控制台，用户初始化管理员账户
2. 进入AI流量入口管理的功能
3. AI服务提供者管理->创建 AI 服务提供者
    1. 大模型供应商(Ollama)->服务名称(`ds1`)->协议(openai/v1)->Ollama 服务主机名(192.168.3.201)->Ollama 服务端口(8880)
    2. 大模型供应商(Ollama)->服务名称(`ds2`)->协议(openai/v1)->Ollama 服务主机名(192.168.3.201)->Ollama 服务端口(8881)
    3. 大模型供应商(Ollama)->服务名称(`ds3`)->协议(openai/v1)->Ollama 服务主机名(192.168.3.201)->Ollama 服务端口(8882)
4. AI路由管理
    1. 名称(`ds-r1`)->路径（/）->选择模型服务(按比例)->目标AI服务(服务名称：DeepSeek-R1-1，请求比例：30 | 服务名称：DeepSeek-R1-2，请求比例：30 | 服务名称：DeepSeek-R1-2，请求比例：40)

#### 测试Higress的API访问

Higress 提供了兼容 OpenAI 数据格式的服务，可以使用 OpenAI 数据格式进行路由的请求。Higress通过路由管理(`ds-r1`的模型)调用会将请求随机路由到一个Ollama服务上去。

> 注意：这里的model名称为自定义的名称(`ds-r1`, `ds1`, `ds2`, `ds3`)

```shell
# `curl`：命令行工具，用于发送 HTTP 请求。
# `-sv`：显示详细交互过程（-v 是详细模式，-s 隐藏进度但保留错误）
# `http://localhost:8080`: 模型服务地址
# `/v1/chat/completions`: OpenAI 兼容的对话补全接口路径
# `-H "Content-Type: application/json"`: 声明请求体为 JSON 格式，确保服务端正确解析数据。
# -d 参数中的 JSON 数据：指定调用的模型名称和对话历史数组
[admin@localhost ~]$ curl -sv http://192.168.3.201:8080/v1/chat/completions \
    -X POST \
    -H 'Content-Type: application/json' \
    -d \
        '{
          "model": "ds-r1",
          "messages": [
            {
              "role": "user",
              "content": "Hello!"
            }
          ]
        }'
```

#### Higress消费者管理

Hitgress可以通过分配多个API Key来管理公司的内部员工。

首先，在控制台将页面切到`消费者管理`，`创建消费者`。

其次，消费者名称(admin)->认证方式(Key Auth)->认证令牌(f236fb02-340b-4119-b97c-95a6315f7964)->令牌来源(Authorization: Bearer ${value})

最后，进行API调用测试

```shell
# `curl`：命令行工具，用于发送 HTTP 请求。
# `-sv`：显示详细交互过程（-v 是详细模式，-s 隐藏进度但保留错误）
# `http://localhost:8080`: 模型服务地址
# `/v1/chat/completions`: OpenAI 兼容的对话补全接口路径
# `-H "Content-Type: application/json"`: 声明请求体为 JSON 格式，确保服务端正确解析数据。
# `-H "Authorization: Bearer f236fb02-340b-4119-b97c-95a6315f7964"`：使用 Bearer 令牌认证，提供身份验证凭证，表明请求者有权访问该接口。
# -d 参数中的 JSON 数据：指定调用的模型名称和对话历史数组
[admin@localhost ~]$ curl -sv http://192.168.3.201:8080/v1/chat/completions \
    -X POST \
    -H 'Content-Type: application/json' \
    -H "Authorization: Bearer f236fb02-340b-4119-b97c-95a6315f7964" \
    -d \
'{
  "model": "ds-r1",
  "messages": [
    {
      "role": "user",
      "content": "Hello!"
    }
  ]
}'
```

#### Token用量观测

Higress 内置了 Prometheus + Grafana 的指标监控套件。可以将页面切换到`AI 控制面板`进行查看。

### LobeChat 的部署

> 由于国内无法访问 DockerHub，使用代理地址访问。

`docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/lobehub/lobe-chat:latest`

#### 创建并运行LobeChat容器

> 由于通过 LobeChat 的 OpenAI 接口对接 Higress，因此需要填写 OPENAI_API_KEY 和 OPENAI_PROXY_URL 两个参数，OPENAI_API_KEY 是上文中我们二次分租创建的 API Key，如果你没有开启二次分租，则可以填写 unused。OPENAI_PROXY_URL 即为 higress 的 HTTP 访问地址。LobeChat 的客户端会通过 3210 端口暴露。

```shell
docker run -d -p 3210:3210   -e OPENAI_API_KEY=e7bb487e-7cd7-43fd-b129-2e943df76ed4   -e OPENAI_PROXY_URL=http://192.168.3.201:8080  -e ACCESS_CODE=lobe66 --name lobe-chat swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/lobehub/lobe-chat
```

#### LobeChat 对话客户端

在浏览器输入 < 公网地址 >:3210 即可打开 LobeChat 对话客户端，然后就可以开始对话了，密码是`lobe66`。

### Cherry Studio的部署

> Cherry Studio的部署很简单，直接下载便携包，启动配置Ollama服务就可以进行使用了。

1. 下载对应版本的Cherry Studio
2. 在 Cherry Studio 中添加 Ollama 作为自定义 AI 服务商：
   1. 打开设置： 在 Cherry Studio 界面左侧导航栏中，点击“设置”（齿轮图标）。
   2. 进入模型服务： 在设置页面中，选择“模型服务”选项卡。
   3. 添加提供商： 点击列表中的 Ollama。
3. 配置 Ollama 服务商
   1. 启用状态：
      1. 确保 Ollama 服务商最右侧的开关已打开，表示已启用。
   2. API 密钥：
      1. Ollama 默认不需要 API 密钥。您可以将此字段留空，或者填写任意内容。
   3. API 地址：
      1. 填写 Ollama 提供的本地 API 地址。
   4. 保持活跃时间：
      1. 此选项是设置会话的保持时间，单位是分钟。如果在设定时间内没有新的对话，Cherry Studio 会自动断开与 Ollama 的连接，释放资源。
   5. 模型管理：
      1. 点击“+ 添加”按钮，手动添加您在 Ollama 中已经下载的模型名称（例如`ds-r1`）。

## 参考

1. Rocky系统命令行界面：[Rocky-9-latest-x86_64-minimal.iso](https://dl.rockylinux.org/pub/rocky/9.5/isos/x86_64/Rocky-9-latest-x86_64-minimal.iso)
2. Rocky系统命令行界面校验文件：[Rocky-9-latest-x86_64-minimal.iso.CHECKSUM](https://dl.rockylinux.org/pub/rocky/9.5/isos/x86_64/Rocky-9-latest-x86_64-minimal.iso.CHECKSUM)
3. Rocky系统图形界面：[Rocky-9-KDE-x86_64-latest.iso](https://dl.rockylinux.org/pub/rocky/9.5/live/x86_64/Rocky-9-KDE-x86_64-latest.iso)
4. Rocky系统图形界面校验文件：[Rocky-9-KDE-x86_64-latest.iso.CHECKSUM](https://dl.rockylinux.org/pub/rocky/9.5/live/x86_64/Rocky-9-KDE-x86_64-latest.iso.CHECKSUM)
5. Rufus工具：[Rufus](https://github.com/pbatard/rufus/releases/download/v4.6/rufus-4.6_x86.exe)
6. CUDA仓库：[cuda rhel9 repos](http://developer.download.nvidia.com/compute/cuda/repos/rhel9)
