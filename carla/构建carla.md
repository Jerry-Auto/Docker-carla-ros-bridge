# CARLA 安装与配置指南

本文档提供两种部署 CARLA 仿真环境的方式：
- 方法一：在宿主机上直接安装并配置 Conda 虚拟环境  
- 方法二：使用 Docker 容器部署 CARLA

文档更新时间：2025年12月

## 目录
- [CARLA 安装与配置指南](#carla-安装与配置指南)
  - [目录](#目录)
  - [方法一：宿主机安装（Conda）](#方法一宿主机安装conda)
    - [1. 下载与导入地图](#1-下载与导入地图)
    - [2. 配置 .egg 文件（Python API）](#2-配置-egg-文件python-api)
    - [3. 示例依赖安装](#3-示例依赖安装)
  - [方法二：使用 Docker 部署 CARLA](#方法二使用-docker-部署-carla)
    - [1. 拉取镜像](#1-拉取镜像)
    - [2. 创建并运行容器（示例）](#2-创建并运行容器示例)
    - [3. 在容器中启动 CARLA](#3-在容器中启动-carla)
  - [同时运行 ROS Bridge 的提示](#同时运行-ros-bridge-的提示)
  - [常见问题与建议](#常见问题与建议)

---

## 方法一：宿主机安装（Conda）

### 1. 下载与导入地图
1. 访问 CARLA 官方仓库并下载预编译包，例如 `CARLA_0.9.15.tar.gz`：  
   https://github.com/carla-simulator/carla  
2. 下载额外地图包，例如 `AdditionalMaps_0.9.15.tar.gz`。  
3. 解压并复制地图包到 Import/：

```bash
tar -xzf CARLA_0.9.15.tar.gz
cp AdditionalMaps_0.9.15.tar.gz CARLA_0.9.15/Import/
```

4. 进入 CARLA 目录并运行导入脚本：

```bash
cd CARLA_0.9.15
./ImportAssets.sh
```

导入完成后，新增地图将在仿真中可用。

---

### 2. 配置 .egg 文件（Python API）
1. 从 ROS-Bridge Releases 下载与当前 Python 版本匹配的 `.egg` 或 `.whl`（示例：`py3.8` 对应 Python 3.8）：  
   https://github.com/carla-simulator/ros-bridge/releases

2. 将 `.egg` 放入 `PythonAPI/carla/dist/` 并解压：

```bash
cd /path/to/CARLA_0.9.15/PythonAPI/carla/dist/
unzip carla-0.9.15-py3.8-linux-x86_64.egg -d carla-0.9.15-py3.8-linux-x86_64
cd carla-0.9.15-py3.8-linux-x86_64
```

3. 如果需要创建可编辑安装，添加 `setup.py`：

```python
# setup.py 示例
from distutils.core import setup
setup(
    name='carla',
    version='0.9.15',
    py_modules=['carla'],
)
```

4. 在 Conda 虚拟环境中安装（确保路径与 Python 版本一致）：

```bash
pip install -e /full/path/to/carla-0.9.15-py3.8-linux-x86_64
```

注意：
- `.egg` 的 `pyX.Y` 必须与当前 Python 版本匹配；否则请下载对应版本或使用 wheel/源码重建。  
- 如果你只有 `.egg` 文件，也可以直接用 `pip install /path/to/file.egg`（仍需匹配 Python 版本）。

---

### 3. 示例依赖安装
进入示例目录（如 `PythonAPI/examples/`）并安装依赖：

```bash
pip install -r requirements.txt
pip install networkx numpy shapely
# Ubuntu/Debian 系统安装 libomp
sudo apt update && sudo apt install -y libomp5
```

---

## 方法二：使用 Docker 部署 CARLA

### 1. 拉取镜像
示例使用国内镜像加速（替换成官方或你需要的版本）：

```bash
docker pull registry.cn-beijing.aliyuncs.com/synkrotron/carla:0.9.14
# 或使用官方镜像/本地私有仓库
```

### 2. 创建并运行容器（示例）
此示例支持 GPU（需安装 NVIDIA Container Toolkit）、X11 GUI（需正确设置 DISPLAY 与 X 权限）：

```bash
docker run --privileged \
  --name carla_container \
  --gpus all \
  --net host \
  -e DISPLAY=$DISPLAY \
  -e SDL_VIDEODRIVER=x11 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -it registry.cn-beijing.aliyuncs.com/synkrotron/carla:0.9.14 /bin/bash
```

说明与建议：
- 将镜像名替换为实际镜像（例如 `registry.cn-beijing.../carla:0.9.14` 或本地 image id）。  
- 若使用 Wayland 或非标准 X server，请根据宿主机环境调整。  
- GPU 支持需要宿主机安装并配置 nvidia-container-toolkit；否则移除 `--gpus all`。

### 3. 在容器中启动 CARLA
容器内启动仿真：

```bash
cd /home/carla
./CarlaUE4.sh
```

若宿主机无法看到窗口，请检查 DISPLAY、X 权限与 X server（以及可能的 Xauthority）。

---

## 同时运行 ROS Bridge 的提示
- 建议将 CARLA 与 ros-bridge 分开部署（不同容器或宿主机），通过网络互联（`--net host` 或自定义 Docker network）进行通信。  
- 参考官方仓库： https://github.com/carla-simulator/ros-bridge

---

## 常见问题与建议
- Python 版本不匹配是常见问题：检查 `.egg` 文件名中的 `pyX.Y` 并与当前 Python 版本一致。  
- GUI 无显示：确认 DISPLAY、X 权限、/tmp/.X11-unix 映射与 Xauthority（如需要使用 xhost +local:root 临时授权）。  
- GPU 无效：确认宿主机驱动、nvidia-container-toolkit 已安装并配置。  
- 缺少库（如 libomp.so.5）：在宿主机或容器内安装相应系统包（如 `libomp5`）。  
- 若需调试网络问题，可在宿主机与容器中使用 `nc` 或 `ros2 topic echo`（根据使用的 ROS 版本）。

---