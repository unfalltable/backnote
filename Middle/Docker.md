---
title: Docker
keywords: [middle]
---

## 简介

- 不同环境的操作系统不同，Docker如何解决的
  - 将用户程序（配置、依赖）和需要用到的系统函数库一起打包（镜像），只要内核相同都可以运行
  - 互相隔离，沙箱机制
- 与虚拟机的差别
  - docker是容器虚拟化，虚拟的是操作系统，虚拟机虚拟的是硬件
  - 虚拟机可以运行不同的操作系统，容器只能运行同一类型的操作系统
  - docker性能更好，硬盘占用小，秒级启动速度，支持上千个容器
- 镜像
  - docker将用户程序（环境、配置、依赖）和需要用到的系统函数库一起打包为一个镜像
- 容器
  - 运行镜像形成的进程就是容器，docker会给容器做隔离，对外不可见
- DockerHub
  - `hub.docker.com`


## 架构

- CS架构
  - 服务端：Docker守护线程（Docker Daemon），负责处理Docker命令，管理镜像、容器
  - 客户端：向Docker发送命令

## 常用命令

| 效果                   | 指令                                  | 可选参数                                       | 参数说明                                                     |
| ---------------------- | ------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 启动docker             | systemctl start docker                |                                                |                                                              |
| 拉取镜像               | docker pull                           |                                                |                                                              |
| 查看镜像               | docker images                         |                                                |                                                              |
| 推送/发布 镜像         | docker push                           |                                                |                                                              |
| 压缩镜像               | docker save                           | -o                                             | 压缩包名称                                                   |
| 解压缩镜像             | docker load                           |                                                |                                                              |
| 重命名镜像             | docker tag 原名 新名                  |                                                |                                                              |
| 查看运行中的容器       | docker ps                             | -a                                             | 查看所有                                                     |
| 删除容器               | docker rm id                          | r m i<br />-f                                  | 删除镜像<br />强制删除                                       |
| 创建容器               | docker run  {相关参数}  镜像名        | -v<br />--name<br />-p<br />-d<br />-e         | 挂载数据卷 数据卷名:容器内路径<br />容器名称<br />端口映射 宿主机 : 容器<br />后台运行<br />环境变量 |
| 进入容器               | docker exec{容器名}                   | -it <br />bash                                 | 允许与容器交互（输入/出）<br />终端交互命令                  |
| 查看容器日志           | docker logs                           | -f                                             | 实时打印日志                                                 |
| 查看容器内部配置信息   | docker inspect                        |                                                |                                                              |
| 关闭/开启/重启容器     | docker stop/start/restart             |                                                |                                                              |
| 将DockerFile构建为镜像 | docker build                          | -t<br /> .                                     | 名字:版本<br />构建dockerfile所在的目录                      |
| 数据卷相关命令         | docker volume                         | create<br />inspect<br />ls<br />prune<br />rm | 创建一个数据卷<br />显示一个或多个数据卷的信息<br />列出所有的volume<br />删除未使用的数据卷<br />删除指定的数据卷 |
| 部署集群               | docker-compose up                     | -d                                             | 后台运行                                                     |
| 容器自动启动           | docker update 容器名 --restart=always |                                                |                                                              |

## 常用操作

- 容器转镜像
  - docker commit 容器id 镜像名称:版本号
  - docker save -o 压缩文件名称 镜像名称:版本号
  - docker load -i 压缩文件名称
- 查看最新的几条日志
  - `docker logs -f -t --tail 条数 模块名`


## 数据卷

- 是一个虚拟目录，指向宿主机文件系统中的目录（/var /lib /docker /volumes / ）
- 分目录挂载和文件挂载
  - 目录挂载：由docker管理，目录路径较长
  - 文件挂载：手动创建，目录清晰，但需要自己管理

## 安装

1. 卸载旧版本

   ```shell
   sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

2. 安装所需软件包

   ```shell
   sudo yum install -y yum-utils \
     device-mapper-persistent-data \
     lvm2
   ```

3. 配置阿里云源

   ```shell
   sudo yum-config-manager \
       --add-repo \
       http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. 安装

   ```shell
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

5. 设置开机自启

   ```shell
   sudo systemctl enable docker
   ```

6. 容器加速服务

   ```shell
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://a3fp19zs.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

   

# DockerFile

## 结构

- 基础镜像层(BaseImage)
  - 包含系统的函数库、环境变量、文件系统
- 入口(EntryPoint)
  - 镜像中应用程序启动的命令
- 层(Layer)
  - 在基础镜像层基础上添加安装包、依赖、配置等，每次操作形成一层

## 指令

- | 关键字     | 作用               | 备注                      |
  | ---------- | ------------------ | ------------------------- |
  | from       | 指定父镜像         | 说明是基于哪个image构建的 |
  | run        | 执行一段命令       |                           |
  | entrypoint | 镜像启动命令       | 入口                      |
  | copy       | 复制文件           | 复制文件到image中         |
  | expose     | 指定容器运行时端口 |                           |
  | env        | 设置环境变量       | 指定环境变量              |

## 步骤

- 创建一个空的文件夹

- 导入jar包，程序jar包，DockerFile

  - DockerFile内容

    - ```dockerfile
      #指定基础镜像
      FROM java:8-alpine
      #复制jar
      COPY ./jar路径 目标路径/xxx.jar
      #暴露端口
      EXPOSE 8081
      #入口 Java项目的启动命令
      ENTRYPOINT java -jar 目标路径/xxx.jar
      ```

# Docker Compose

## 简介

- 通过compose文件快速部署分布式应用

## 安装

- ```shell
  #下载
  curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-'uname -s'-'uname -m' -o /usr/local/bin/docker-compose
  #设置权限
  chmod +x /usr/local/bin/docker-compose
  #查看版本信息
  docker-compose -version
  ```

## 部署Nginx+SpringBoot项目

- ```shell
  #创建目录并进入
  mkdir ~/docker-compose
  cd ~/docker-compose
  ```

- ```yaml
  #编写文件 docker-compose.yml
  version: ''
  services:
  	nginx:
  		image: nginx
      	ports: 
      		- 80:80
      	links:
      		- qpp
      	volumes:
      		- ./nginx/conf.d:/etc/nginx/conf.d
      app:
      	image: app
      	expose:
      		- "8080"
  ```

- ```shell
  #创建/nginx/conf.d 目录
  mkdir -p ./nginx/conf.d
  ```

- ```xml
  #创建并编写xxx.conf
  server{
  	listen 80;
  	access_log off;
  
  	location / {
  		proxy_pass http://app:8080;
  	}
  }
  ```

- ```shell
  #在~/docker-compose目录下启动容器
  docker-compose up
  #测试
  ```

## 部署微服务集群

- 编写各个微服务的dockerfile，并且和微服务的jar包放在同一级目录下
  - mysql需要打包配置和数据
- 编写docker-compose.yml文件
- 修改各个微服务内服务调用时的ip，用服务名替代
- 将docker-compose和所有微服务的dockerfile和jar打包到服务器上

# 私有仓库

## 搭建

- 使用docker-compose部署具有图形化界面的DockerRegistry

  ```yaml
  version: '3.0'
  services:
  	registry:
  		image: registry
  		volumes: 
  			- ./registry-data:/var/lib/registry
  	ui:
  		image: joxit/docker-registry-ui:static
  		port:
  			- 8080:80
  		environment:
  			- REGISTRY_TITLE=
  			- REGISTRY_URL=http://registry:5000
  		depends_on:
  			- registry
  ```

- 配置Docker信任地址，私服一般是http协议，docker默认不信任

  ```shell
  #打开要修改的文件
  vi /etc/docker/daemon.json
  #添加内容
  "insecure-registries":["http:..."]
  #重加载
  systemctl daemon-reload
  #重启docker
  systemctl restart docker
  ```

## 使用

- 推送镜像需要先tag（重命名镜像）

# 整合

## Redis容器

### 安装

1. 拉取

   ```shell
   docker pull redis
   ```

2. 启动

   ```shell
   docker run -p 6379:6379  --name redis\
   	-v /mydata/redis/data:/data\ #数据文件挂载
   	-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf\ #配置文件挂载
   	-d redis redis-server /etc/redis/redis.conf #后台运行
   	--appendonly yes #开启aof持久化
   ```

3. 进入容器

   ```shell
   docker exec -it redis redis-cli
   ```

## Nginx容器

- `docker run --name nginx -p 80:80 -v html:/usr/share/nginx/html -d nginx`
  - 容器内路径可以在dockerhub上查看

## MySQL容器

### 安装

1. 拉取

   ```shell
   docker pull mysql
   ```

2. 启动

   ```shell
   docker run -p 3306:3306 --name mysql\
   	-v /mydata/mysql/log:/var/log/mysql\
   	-v /mydata/mysql/data:/var/lib/mysql\ #数据文件挂载
   	-v /mydata/mysql/conf:/etc/mysql/conf.d/hmy.cnf #配置文件挂载
   	-e MYSQL_ROOT_PASSWORD=123456\
   	-d mysql #后台运行
   ```

   - 挂载是文件夹需要存在

3. 进入容器

   ```shell
   docker exec -it mysql bash
   ```

   
