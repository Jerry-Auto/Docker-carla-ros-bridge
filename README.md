# Docker-carla-ros-bridge
使用docker分别部署carla和ros-bridge仿真环境的办法
分别在carla和ros-brodge文件夹下详细说明了构建方法

这里给出最简版本

1.二进制版本

```bash
docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/carla_bin:914

sudo docker run -dit \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=all,utility,vulkan,graphics,compute \
  -e VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json \
  -e XAUTHORITY="$HOME/.Xauthority" \
  -e DISPLAY="$DISPLAY" \
  -e LANG=C.UTF-8 \
  -e LC_ALL=C.UTF-8 \
  --name carla914 \
  --net host \
  -v /dev:/dev \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v /usr/share/vulkan/icd.d/nvidia_icd.json:/usr/share/vulkan/icd.d/nvidia_icd.json:ro \
  -v "$HOME/.Xauthority":"$HOME/.Xauthority":ro \
  -v /usr/share/fonts:/usr/share/fonts:ro \
  -v "$HOME/.local/share/fonts":"$HOME/.local/share/fonts":ro \
  crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/carla_bin:914 \
  /bin/bash
```
进入容器后
```bash
fc-cache -fv
su carla
GUI
```

2.ROS2-bridge

```bash
docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros2-bridge:foxy

sudo docker run -dit \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=all \
  --name ros2_bridge \
  --net host \
  -v /dev:/dev \
  -v $HOME:$HOME \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  -w $HOME \
  crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros2-bridge:foxy /bin/bash

```

3.源码编译版本

下载APP文件，网盘链接如下：

https://www.123865.com/s/IR8MTd-UJev3

```bash
docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/carla_dev_env:914

sudo docker run -dit \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=all,utility,vulkan,graphics,compute \
  -e VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json \
  -e XAUTHORITY="$HOME/.Xauthority" \
  -e DISPLAY="$DISPLAY" \
  -e LANG=C.UTF-8 \
  -e LC_ALL=C.UTF-8 \
  --name carla \
  --net host \
  -v /dev:/dev \
  -v "$HOME":"$HOME" \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v /usr/share/vulkan/icd.d/nvidia_icd.json:/usr/share/vulkan/icd.d/nvidia_icd.json:ro \
  -v "$HOME/.Xauthority":"$HOME/.Xauthority":ro \
  -v /usr/share/fonts:/usr/share/fonts:ro \
  -v "$HOME/.local/share/fonts":"$HOME/.local/share/fonts":ro \
  -w "$HOME" \
  crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/carla_dev_env:914 \
  /bin/bash

fc-cache -fv
ln -sfn /home/yourname/path/to/carla_env /carla

```