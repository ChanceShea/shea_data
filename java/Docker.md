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
