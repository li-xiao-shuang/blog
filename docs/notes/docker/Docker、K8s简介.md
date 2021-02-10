# Docker

## **Docker原理**

![https://img.draveness.me/2017-11-30-docker-core-techs.png](https://img.draveness.me/2017-11-30-docker-core-techs.png)

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

**Namespace机制：**

容器内看不到宿主机的其他进程，通过Linux的Namespace机制实现环境隔离。过 Namespace 技术，我们实现了容器和容器间，容器与宿主机之间的隔离

**容器的限制：Cgroups：**

`Cgroups` 就是 Linux 内核中用来为进程设置资源的一个技术。

Linux Cgroups 全称是 Linux Control Group，主要的作用就是限制进程组使用的资源上限，包括 CPU，内存，磁盘，网络带宽。还可以对进程进行优先级设置，审计，挂起和恢复等操作

## **Docker 的主要用途，目前有三大类**

**（1）提供一次性的环境。**比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

**（2）提供弹性的云服务。**因为 Docker 容器可以随开随关，很适合动态扩容和缩容。

**（3）组建微服务架构。**通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## **Docker安装**

- yum install docker
- docker -v
- systemctl start docker

## **常用命令**

- docker version 查看docker版本
- docker image list 查看本地镜像列表
- docker ps 查看运行中的镜像进程
- docker search xx 查找镜像
- docker pull xx 拉取镜像到本地
- docker run xx 启动镜像
- docker stop xx 关闭镜像
- docker exec -it xx /bin/bash 进入容器命令

## **Dockerfile怎么编写**

- 先创建一个Dockerfile文件，写入如下

  ![https://tva1.sinaimg.cn/large/0081Kckwly1gm02zfjkxyj30q608ygn1.jpg](https://tva1.sinaimg.cn/large/0081Kckwly1gm02zfjkxyj30q608ygn1.jpg)

- `FROM` :需要构建镜像的项目所需要依赖的`基础镜像`，`SpringBoot`项目是跑在`JDK`之上的

- `VOLUME` :定义匿名数据卷，容器在运行的时候，会将数据写入到这个数据卷中，这里设置为一个临时目录

- `ADD` ：将target目录下的springboot-docker-0.0.1-SNAPSHOT.jar`包添加到`docker容器中，并将名称进行修改为docker.jar

- `RUN`:执行后其后面的命令

- `ENTRYPOINT`;在容器启动之前的预定义执行脚本命令

- `EXPOSE` : 启动的端口

- docker build -t springboot-docker/springboot-docker:1.0 . 构建镜像

## **Docker 私有仓库搭建**

1. docker pull registry

2. docker run -d -p 5000:5000 --name=docker_registry --restart=always --privileged=true -v /usr/local/docker_registry:/var/lib/registry

   docker.io/registry

    - -p 5000:5000 端口
    - -d 在后台运行
    - --name=jackspeedregistry 运行的容器名称
    - --restart=always 自动重启
    - --privileged=true centos7中的安全模块selinux把权限禁止了，加上这行是给容器增加执行权限
    - v /usr/local/docker_registry:/usr/local/docker_registry 把主机的/usr/local/docker_registry 目录挂载到registry容器的/usr/local/docker_registry目录下，假如有删除容器操作，我们的镜像也不会被删除 [docker.io/registry](http://docker.io/registry) 镜像名称查看启动的容器

# **Kubernetes 集群**

![https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

**Kubernetes 协调一个高可用计算机集群，每个计算机作为独立单元互相连接工作。** **Kubernetes 以更高效的方式跨集群自动分发和调度应用容器。** Kubernetes 是一个开源平台，并且可应用于生产环境。

一个 Kubernetes 集群包含两种类型的资源:

- **Master** 调度整个集群
- **Nodes** 负责运行应用

**Master 负责管理整个集群。** Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。

**Node 是一个虚拟机或者物理机，它在 Kubernetes 集群中充当工作机器的角色** 每个Node都有 Kubelet , 它管理 Node 而且是 Node 与 Master 通信的代理。 Node 还应该具有用于处理容器操作的工具，例如 Docker 或 rkt 。处理生产级流量的 Kubernetes 集群至少应具有三个 Node 。

![https://www.redhat.com/cms/managed-files/kubernetes-diagram-2-824x437.png](https://www.redhat.com/cms/managed-files/kubernetes-diagram-2-824x437.png)