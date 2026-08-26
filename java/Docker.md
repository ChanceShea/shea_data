## Docker底层依托Linux怎么实现资源隔离
- **基于Namspace的视图隔离**：Docker利用Linux命名空间来实现不同容器之间的隔离。每个容器都运行在自己的一组命名空间中，包括PID、网络、挂载点、IPC（进程间通信）等。这样，容器中的进程只能看到自己所在的命名空间内的进程，而不会影响其他容器中的进程
- **基于cgroups的资源隔离**：cgroups是Linux内核的一个功能，允许在进程组之间分配、限制和优先处理系统资源，如CPU、内存和磁盘IO。它们提供了一种机制，用于管理和隔离进程集合的资源使用，有助于资源限制、工作负载隔离，以及在不同进程组之间进行资源优先处理
## Docker和虚拟机有什么区别
**核心区别**：虚拟机虚拟化的是操作系统，Docker虚拟化的是进程
- **虚拟机**：通过Hypervisor在宿主机上模拟一整套硬件，每台VM都跑着一个完整的Guest OS内核，隔离性极强但开销大——启动需要几十秒、镜像几个GB、单实例内存占用GB级别
- **Docker容器**：所有容器共享宿主机的Linux内核，通过Namespace做视图隔离、cgroups做资源限制，本质上容器就是一组被隔离起来的特殊进程。启动秒级、镜像几十到几百MB、内存开销MB级别
## Docker的核心组件
- **镜像（Image）**：只读的模板，包含了运行一个应用所需要的代码、依赖、环境、变量、配置等，相当于安装包
- **容器（Container）**：镜像的运行时实例，可以启动、停止、删除，是一个独立隔离的进程组，相当于安装后运行起来的程序
- **仓库（Registry）**：存储和分发镜像的地方，分为公有仓库（Docker Hub）和私有仓库（如阿里云ACR、腾讯云TCR），类似于应用商店
从Registry拉取镜像->由镜像启动容器->容器运行应用
镜像是静态的模板，容器是镜像的动态运行实例。就类似于**类和对象**。类用于定义应用的结构和行为，是只读的。对象是类的具体实例
## Docker镜像的分层原理
Docker镜像采用分层只读的结构，底层基于**UnionFS（联合文件系统）**，现代Docker主要使用`overlay2`作为默认的存储驱动
Dockerfile里每一条指令都会产生一个新的镜像层，这些层都是只读的。例如
```dockerfile
FROM ubuntu:22.04    # 层1
RUN apt-get update   # 层2
COPY app /app        # 层3
CMD ["./app"]        # 层4
```
当容器启动时，Docker会在所有只读层之上再叠加一层可写层（容器层），容器对任何文件的修改都发生在这一层，通过**Copy-on-Write（写时复制）** 机制实现
分层的好处
- **共享与复用**：多个镜像可以共享相同的底层，节省存储和网络传输
- **构建缓存**：重新构建是，只要某一层没变就直接复用缓存，速度极快
- **增量分发**：`docker pull`只会下载变化的那一层
Dockerfile最佳实践里有一条重要原则：**把不常变的层（如安装依赖）放在常变的层（如COPY源码）之前**，这样缓存命中率更高、构建更快
## Dockerfile常用指令
- FROM：指定基础镜像，必须是Dockerfile的第一条指令
- WORKDIR：设置容器内的工作目录，相当于cd
- COPY/ADD：把宿主机的文件拷贝到镜像里
- RUN：在构建阶段执行命令（如安装依赖），每条RUN都会产生一个新层
- ENV：设置环境变量（运行时也能看到）
- ARG：构建时的参数，只在`docker build`阶段有效
- EXPOSE：声明容器要监听的端口（只是声明，不会真的开放端口）
- VOLUME：声明数据卷挂载点
- CMD/ENTRYPOINT：指定容器启动时默认要运行的命令
- USER：切换到非root用户运行
以下是一个SpringBoot应用的Dockerfile：
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```
**CMD和ENTRYPOINT的区别**
- CMD：提供默认命令，可以被`docker run`后面跟的参数完全覆盖
- ENTRYPOINT：设置容器的主命令，不会被`docker run`覆盖，后面跟的参数会追加到ENTRYPOINT后面
**COPY和ADD的区别**
两者都是把文件从宿主机拷贝到镜像中，但ADD多了两个功能
- **自动解压**：如果源文件是`tar.gz`之类的压缩包，ADD会自动解压到目标目录
- **支持URL**：ADD可以直接从URL下载远程文件到镜像里
虽然ADD的功能更多，但是官方和Docker最佳实践都推荐优先使用COPY
- ADD的功能太多反而导致一些步骤不透明，容易出bug
- 如果需要下载远程文件，推荐使用`RUN curl -fLO https://...`明确写出来，行为更可控
- 如果需要解压，在RUN里显示处理更清晰
## 多阶段构建
多阶段构建是Docker 17.05引入的一个重要特性，允许在一个Dockerfile里写多个FROM，每个FROM开启一个构建阶段，后面的阶段可以从前面的阶段拷贝文件，但最终只有最后一个阶段的镜像会被保留
核心价值：**把构建环境和运行环境分开，大幅度减小最终镜像体积**
例如
```dockerfile
FROM golang:1.21
COPY ./src
RUN cd /src && go build -o /app
CMD ["/app"]
```
上述镜像中包含了整个GO编译器、依赖、源码，可能有800MB+
对于多阶段构建
```dockerfile
# 第一阶段：构建
FROM golang:1.21 AS builder
WORKDIR /src
COPY ..
RUN go build -o app
# 第二阶段：运行
FROM alpine:3.18
COPY --from=builder /src/app /app
CMD ["/app"]
```
最终镜像只基于alpine（5MB）加上编译好的二进制，可能只有15MB，瘦身50倍。Java，Node.js，前端项目都是和这种模式
## 如何减小Docker镜像体积
1. **选用小的基础镜像**：用`alpine`代替`ubuntu`，用`openjdk:17-jre-slim`代替`openjdk:17`
2. **多阶段构建**：构建和运行分离，最终镜像只保留运行所需的产物
3. **合并RUN指令**：每条RUN都会产生一个新层，用&&把多个命令合并成一条RUN，减少层数
4. **清理构建中间产物**：在同一条RUN里装完包就立刻清理缓存
5. **使用`.dockerignore`**：像`.gitignore`一样，排除不需要COPY进镜像的文件
6. **避免在镜像里装无关的调试工具**：调试工具应该在需要时`docker exec`临时装，不该塞进生产镜像
## Docker容器数据持久化方式
容器本身是用完即扔的，想持久化数据必须使用外部存储
- **Volume（数据卷）**：Docker官方推荐。由Docker管理，默认存储在宿主机的`/var/lib/docker/volumes`下，可以用`docker volume`系列命令管理，最通用也最推荐
```sh
docker run -v mydata:/app/data myimage
```
- **Bind Mount（目录挂载）**：把宿主机的任意路径直接挂进容器，灵活但耦合宿主机路径、不利于迁移，适合开发阶段的热更新调试
```sh
docker run -v /home/user/code:/app myimage
```
- **tmpfs**：挂载到宿主机内存里，容器停止数据就没了，适合存放敏感临时数据
## Docker的网络模式有哪些
- **bridge（默认）**：Docker创建一个docker0虚拟网桥，每个容器分配独立IP，容器之间通过网桥通信，访问外网通过NAT。单机多容器的默认场景
- **host**：容器直接使用宿主机的网络栈，不做隔离。性能最好但会和宿主机抢端口，无法在一台机器上起多个监听同一端口的容器
- **container**：和指定的另一容器共享同一个网络栈（K8s里Pod内多个容器就是类似机制）
- **overlay**：跨宿主机的网络，通常在`Docker Swarm`或`K8s`里使用
## 同一个宿主机上的容器之间如何通信
- 默认bridge网络（不推荐）：通过容器的IP直接访问，但IP不固定，也不支持容器名DNS解析
- 自定义bridge网络（推荐）：自己创建一个网络，加入同一网络的容器之间可以直接用容器名作为DNS名互相访问
```sh
docker network create mynet
docker run -d --name redis --network mynet redis
docker run -d --name app --network mynet myapp
```
- host模式：所有容器共享宿主机网络，直接通过`localhost:port`访问，但失去了端口隔离
- 通过Docker Compose：Compose会自动为一个project创建一个bridge网络，service名就是容器名，互相访问非常方便
## Docker Compose
Docker Compose是Docker官方提供的单机容器编排工具，用一个YAML文件定义一组相关服务，然后一条命令启动或停止整组服务
例如：本地开发或小型单机部署一套“Web应用+数据库+缓存”的组合
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
	  - redis
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - db_data:/var/lib/mysql
  redis:
    images: redis:7
volumes:
  db_data:
```
**常用命令**
- `docker compose up -d`：后台启动所有服务
- `docker compose down`：停止并删除所有服务
- `docker compose logs -f`：实时查看日志
- `docker compose ps`：查看服务状态
## Docker常用命令
- `docker pull <image>`：拉取镜像
- `docker images`：列出本地镜像
- `docker rmi <image>`：删除镜像
- `docker build -t name:tag .`：基于当前目录的Dockerfile构建镜像
- `docker run -d -p 8080:80 --name web nginx`：启动一个后台容器
- `docker ps`：查看运行中的容器（加`-a`参数查看所有容器）
- `docker rm <container>`：删除容器
- `docker stop/start/restart <container>`：停止/启动/重启容器
- `docker exec -it <container> bash`：进入容器
- `docker inspect <container>`：查看容器详细信息（JSON格式）
- `docker stats`：查看容器资源占用
- `docker system df`：查看Docker占用了多少磁盘
- `docker system prune`：清理无用资源
## 容器启动后立刻退出，如何排查
1. 先看日志，`docker logs <container>`或`docker logs --tail 100 <container>`或`docker logs -f <container>`，大部分问题从日志中可以直接看出来
2. 看退出码，`docker ps -a`的STATUS列会显示`Exited(x)`的退出码。0是正常退出，137通常是OOM被杀，139是段错误，其他非0就是应用报错
3. 思考PID=1的问题，Docker容器里主进程必须是前台运行的，如果启动命令是一个后台化的程序（比如写成`nginx`而不是`nginx -g 'daemon off;'`），主进程启动完立刻退出，容器也就跟着结束了
4. 临时用shell进去调试，如果容器一启动就挂，可以临时把`ENTRYPOINT`覆盖成shell进去查看
```sh
docker run -it --entrypoint sh <image>
```
5. 检查资源限制，是否因为cgroups内存限制太小被OOM killer杀掉（docker inspect能看到`OOMKilled:true`）
## `docker commit`和`docker build`的区别
两者都能得到一个新镜像，但是使用场景和推荐度完全不同
- `docker commit <container> <image>`：把一个正在运行的容器的当前状态打包成镜像，相当于“快照”。过程是手动的、隐式的，镜像里发生了什么只有操作者自己知道
- `docker build -f Dockerfile -t <image> .`：根据Dockerfile脚本自动构建镜像。过程是可复现、可追溯、可版本控制的
**生产环境中一定要使用`docker build`**
- Dockerfile可以提交到Git，方便团队协作和代码审查
- 构建过程完全自动化，随时可以从代码重新构建出一模一样的镜像
- 镜像层结构清晰，每一层对应一条指令，便于排查
## `docker run`的常用参数
- `-d`：后台运行
- `-it`：交互式+分配终端，通常一起使用
- `-p 8080:80`：端口映射（宿主机:容器）
- `-v /host:/container`或`-v volname:/container`：目录/数据卷挂载
- `--name web`：给容器起个名字
- `--network mynet`：指定容器网络
- `-e KEY=VALUE`：设置环境变量
- `--rm`：容器停止后自动删除（适合一次性任务）
- `--restart always/unless-stopped`：自动重启策略
- `-m 512m --cpus=1`：内存/CPU资源限制
- `-u 1000:1000`：以非root用户运行
