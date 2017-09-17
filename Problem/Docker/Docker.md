# Docker容器内代理服务器的设定

sudo docker run -e http_proxy=http://proxy.example.com:8888/ -it debian:wheezy
另外，也可以在容器启动后，在容器中的命令行来设定环境变量

> $ sudo docker run -it debian:wheezy
> root@7cb147891556:/# export http_proxy=http://proxy.example.com:8888/
> root@7cb147891556:/#

假如必须使用特定的PROXY的话，那么在Dockerfile中可以直接填写代理服务器信心。
> FROM debian:wheezy
>  ENV http_proxy http://proxy.example.com:8888/
>  ENV https_proxy http://proxy.example.com:8888/
>  RUN apt-get update && apt-get install python3


## Docker加速器
申请DaoCloud用户 https://dashboard.daocloud.io/packages/411e98c1-f84c-4193-937c-39331d9bfe98
在DaoCloud中选择Docker Hub后，查找自己所需的镜像
而后点击拉取后，显示详细的步骤

## Cannot connect to the Docker daemon. Is the docker daemon running on this host?
run the daemon with the following command:
sudo nohup dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &