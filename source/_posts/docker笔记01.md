title: docker笔记01
date: 2019-04-17 18:16:56
tags:
- docker
categories: docker
---

### 安装 Docker
Mac 上的安装
```
brew cask install docker
```

安装完成之后，执行 hello-world 试一下:

```
$ docker run hello-world

```


```

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### 启动Nginx服务
```
docker run -d -p 80:80 --restart=always  nginx:latest
```

> `-p` 表示宿主机IP:容器IP
> `--restart` 重启模式，设置 always，每次启动 docker 都会启动 nginx 容器

### 进入容器
```
docker exec -it 4591552a4185 bash
```
参数说明：
> `exec` 对容器操作
> `-it` 分配伪tty
> `4591552a4185 `容器ID
> `bash`交互程序为bash

Docker 提供数据挂载的功能，即可以指定容器内的某些路径映射到宿主机器上，修改命令，添加 -v 参数，启动新的容器。

```
docker run -d -p 80:80  -v ~/docker-demo/nginx-htmls:/usr/share/nginx/html/ --restart=always  nginx:latest
```

> 启动成功之后，docker 会帮你生成目录 ~/docker-demo/nginx-htmls

### 停止运行
```
docker stop 4591552a4185
```

### 删除容器
```
docker rm 4591552a4185
```

### 分配IP给宿主机

```
sudo ifconfig lo0 alias 192.168.64.0/24
```
访问 `http://192.168.64.0:8080` 