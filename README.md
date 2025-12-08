# Docker-carla-ros-bridge
使用docker分别部署carla和ros-bridge仿真环境的办法
分别在carla和ros-brodge文件夹下详细说明了构建方法

这里给出最简版本

```bash
docker pull registry.cn-beijing.aliyuncs.com/synkrotron/carla:0.9.14

docker run --privileged \
  --name carla914 \
  --gpus all \
  --net host \
  -e DISPLAY=$DISPLAY \
  -e SDL_VIDEODRIVER=x11 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -it registry.cn-beijing.aliyuncs.com/synkrotron/carla:0.9.14 /bin/bash

docker pull crpi-d390lxwlwm12oflr.cn-shanghai.personal.cr.aliyuncs.com/carla_env/ros2-bridge:foxy

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
/home/abc改成自己的