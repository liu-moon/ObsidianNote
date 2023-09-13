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
要在windows中连接执行


创建dockerfile
```shell
mkdir docker
touch Dockerfile
```
安装插件：docker


build
docker build -t r1_noetic_from_file .
