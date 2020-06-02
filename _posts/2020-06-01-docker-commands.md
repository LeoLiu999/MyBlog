---
layout: post
title:  docker常用命令
date:   2020-06-01
categories: docker
permalink : docker/commands.html
description: docker
show : true
---

```shell
docker --version #查看版本
docker search xxx #搜索镜像
docker pull xxx #从镜像仓库下载镜像
docker build  -t xx -f /xxx/Dockerfile . #通过dockerfile创建镜像
docker push user/image #发布docker镜像
docker run [OPTIONS] IMAGE [COMMAND] [ARG...] #运行镜像
docker run -it -v /wwwroot/docker/swoole-php  --name myswoole -p  8081:9501 phpswoole/swoole  #运行phpswoole -v指定目录 -it以交互模式运行容器 -p 主机端口：容器端口映射 
docker attach xxx #链接到正在运行的容器
docker ps #可以查看所有正在运行中的容器列表，
docker inspect versionid #可以查看更详细的关于某一个容器的信息。
docker commit versionid xxx #保存对容器的修改
docker images #查看镜像
docker rmi  IMAGE（镜像ID）#删除镜像
docker stop  ContainerID #停止运行
docker rm  ContainerID #停止运行后可以删除
docker start  containerId #启动
docker restart containerId 重启
docker logs --tail=50 -t -f  containerId  #查看日志
docker logs -t  57e919bac666   >> logs_error.txt #将日志输出到文件
docker exec -it containerId -sh #进入docker查看
docker save -o web-qb_v1.0.3.tar web-qb:v1.0.3 #导出docker

```

