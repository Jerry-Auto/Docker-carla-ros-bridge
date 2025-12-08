# CARLA + ROS Bridge 构建与部署（简明）

说明：本指南以 ROS 2 Foxy 为主。提供两种方式：
- 方法 A：从基础 ROS 容器或宿主机源码搭建（适合开发/定制）。  
- 方法 B：直接拉取已构建好的镜像（开箱即用）。

## 目录
- 准备工作  
- 方法 A：源码搭建（详细步骤）  
- 方法 B：直接拉镜像（快速启动）  
- 测试与验证  
- 常见问题与排查要点

---

## 准备工作
- 确认 CARLA 版本（示例：0.9.15），并下载对应 Python `.egg`/`.whl`（`pyX.Y` 要匹配）。  
- 若需 GPU，宿主机需安装 NVIDIA 驱动与 nvidia-container-toolkit。  
- 建议工作目录：`~/carla-ros-bridge/`，包含 `PythonAPI/` 与 `src/ros-bridge/`。

---

## 方法 A：源码搭建（详细）
### 1. 创建 ROS 容器（Foxy 示例）
```bash
sudo docker run -dit \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=all \
  --name ros2_foxy_gui \
  --net host \
  -v /dev:/dev \
  -v ~/ros2_foxy_ws:/root/ros2_ws \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  osrf/ros:foxy-desktop
```

### 2. 拷贝/挂载 CARLA PythonAPI
推荐容器内目录结构：
```
~/carla-ros-bridge/
  ├─ PythonAPI/    # CARLA 的 PythonAPI
  └─ src/ros-bridge/
```
示例将宿主机 CARLA PythonAPI 拷贝到容器：
```bash
# 从 CARLA 容器拷贝到宿主机
docker cp carla_container:/path/to/CARLA/PythonAPI ~/PythonAPI

# 再从宿主机拷贝到 ros-bridge 容器
docker cp ~/PythonAPI ros_bridge_container:~/carla-ros-bridge/PythonAPI
```

### 3. 安装 .egg/.whl（容器内）
```bash
# 把下载好的 egg/whl 复制到容器的 dist 目录，然后：
cd ~/carla-ros-bridge/PythonAPI/carla/dist/
unzip carla-0.9.15-py3.8-linux-x86_64.egg -d carla-0.9.15-py3.8-linux-x86_64
cd carla-0.9.15-py3.8-linux-x86_64

# 创建 setup.py（可编辑安装）
cat > setup.py <<'EOF'
from distutils.core import setup
setup(
  name='carla',
  version='0.9.15',
  py_modules=['carla'],
)
EOF

pip3 install -e ~/carla-ros-bridge/PythonAPI/carla/dist/carla-0.9.15-py3.8-linux-x86_64
```
注意：`py3.8` 必须与容器内 Python 版本一致。

### 4. 克隆并构建 ros-bridge
```bash
mkdir -p ~/carla-ros-bridge && cd ~/carla-ros-bridge
git clone --recurse-submodules https://github.com/carla-simulator/ros-bridge.git src/ros-bridge

# 然后关键的来了，在克隆下来的carla-ros-bridge/src/ros-bridge/carla_ros_bridge/src/carla_ros_bridge文件夹
# 里面有一个文件CARLA_VERSION，里面就一行，初始是0.9.13，修改成0.9.15（你的版本）

sed -i 's/0.9.13/0.9.15/' src/ros-bridge/carla_ros_bridge/src/carla_ros_bridge/CARLA_VERSION

colcon build
source install/setup.bash
# 可加入 ~/.bashrc
echo "source $(pwd)/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 5. examples 环境与 automatic_control
```bash
cd ~/carla-ros-bridge/PythonAPI/examples
pip3 install -r requirements.txt
pip3 install networkx numpy shapely
sudo apt update && sudo apt install -y libomp5

# 设置 GUI 环境变量（写入 bashrc）
echo -e "\n# CARLA/Pygame GUI\nexport DISPLAY=\$DISPLAY\nexport SDL_VIDEODRIVER=x11\nexport SDL_AUDIODRIVER=dummy" >> ~/.bashrc
source ~/.bashrc

# 在容器内添加 host.docker.internal 映射（需 sudo）
echo "$(route -n | awk '/^0.0.0.0/ {print $2}')" | xargs -I{} sudo bash -c "echo {} host.docker.internal >> /etc/hosts"

# 运行示例
python3 ./automatic_control.py --host host.docker.internal --port 2000

# 如果嫌后面的参数--host host.docker.internal --port 2000麻烦，
# 修改一下automatoc_control.py的配置信息(在main函数里)
# 将：
argparser.add_argument(
    '--host',
    metavar='H',
    default='127.0.0.1',
    help='IP of the host server (default: 127.0.0.1)')
# 改为：
argparser.add_argument(
    '--host',
    metavar='H',
    default='host.docker.internal',
    help='IP of the host server (default: host.docker.internal)')
    
cd /home/abc/AppDisk/carla_ros/CARLA_0.9.15/PythonAPI/example
python3 ./automatic_control.py
```

---

## 方法 B：直接拉镜像（快速）
镜像通常已包含构建产物与依赖，运行时只需传入必要参数。

```bash
docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros2-bridge:foxy
# 如果需要，还有noetic版本的
docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros-noetic-bridge:noetic

sudo docker run -dit \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=all \
  --name ros2_bridge \
  --net host \
  -v /dev:/dev \
  -v /home/abc:/home/abc \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  -w /home/abc \
  crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros2-bridge:foxy /bin/bash
```
要点：无需额外构建；如需访问宿主机 CARLA PythonAPI，可在运行时挂载对应目录。

---

## 测试与验证
1. 启动 CARLA 仿真（宿主或 CARLA 容器）：
```bash
cd /path/to/CARLA && ./CarlaUE4.sh
```
2. 在 ros-bridge 环境启动 bridge：
```bash
ros2 launch carla_ros_bridge carla_ros_bridge.launch.py
# 或
ros2 launch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch.py
```
3. 可视化：运行 `rviz2` 订阅话题。 
    
4. 示例：运行 `automatic_control.py` 验证 vehicle 控制。
```bash
cd /home/abc/AppDisk/carla_ros/CARLA_0.9.15/PythonAPI/example
python3 ./automatic_control.py
```
---

## 常见问题与排查
- .egg/.whl 与 Python 版本不匹配：确认文件名中的 `pyX.Y`。  
- GUI 无显示：检查 `DISPLAY`、`/tmp/.X11-unix` 挂载与 `xhost` 权限。  
- GPU 无效：确认宿主驱动与 `nvidia-container-toolkit` 已安装。  
- 连接失败：检查 CARLA 端口（默认 2000）、防火墙与 host 映射。  
- 构建失败：查看 `colcon` 日志并确认已 source ROS 环境。

---