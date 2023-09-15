windows安装的软件：
- docker
- xlaunch
- ros docker images osrf
安装完foxy-desktop版本之后，可以通过`docker images`命令来进行查看
通过镜像来创建容器：
`docker run -it osrf/ros:foxy-desktop`
查看版本todo
`lsb_release -a`
查看正在运行的容器
`docker ps`
在windows的命令行中使用
`docker exec -it f7a1c0d79de0 bash`


配置foxy版本
```shell
docker run --name r2_foxy -it osrf/ros:foxy-desktop
```

如果想要运行gui版本
```shell
docker run --name r2_foxy -e DISPLAY=host.docker.internal:0.0 -it osrf/ros:foxy-desktop
```

摄像头：
```shell
docker run --name r2_humble_with_cam -e DISPLAY=host.docker.internal:0.0 -it --device=\Device\000000bb osrf/ros:humble-desktop-full
```

要在windows中连接执行


我的摄像头的地址是 /dev/video0



创建dockerfile
```shell
mkdir docker
touch Dockerfile
```
安装插件：docker


build
docker build -t r1_noetic_from_file .



首先要source一下
source opt/ros/foxy/setup.bash




首先拉取镜像
```shell
docker pull osrf/ros:humble-desktop-full
```


docker run --name linux_test_con -it linux_test




## 创建自己的dockerfile
### windows
mkdir docker
touch Dockerfile

FROM osrf/ros:humble-desktop-full

RUN apt-get update
RUN apt-get install -y git && apt-get install -y python3-pip

RUN mkdir -p ~/dev_ws/src && cd ~/dev_ws/src

RUN git clone https://gitee.com/guyuehome/ros2_21_tutorials.git

RUN echo "ALL DONE !"



运行dockerfile

cd docker
docker build -t humble_from_file .

这样就生成了一个名为：humble_from_file的镜像
然后通过以下命令创建容器
docker run -it humble_from_file



### ubuntu
查看镜像
docker images


docker build -t humble_from_file

docker run --name humble_container -it humble_from_file



使用gui

xhost local:root

XAUTH=/tmp/.docker.xauth

docker run -it \
	--name=<name> \
	--env="DISPLAY=$DISPLAY" \
	--env="QT_X11_NO_MITSHM=1" \
	--volume="tmp/.X11-unix:/tmp/.X11-unix:rw" \
	--env="XAUTHORITY=$XAUTH" \
	--volume="$XAUTH:$XAUTH" \
	--net=host \
	--privileged \
	osrf/ros:humble-desktop-full \
	bash

echo "Done."


chomd +x *
./run_docker.bash
记得修改name
