# 理论

**Docker官网**：https://www.docker.com

**准备**：配置2C2G以上Linux服务器一台，云服务器、虚拟机均可。

## 虚拟机与容器

**抛出背景**

在企业中，服务器是必不可少的一种硬件设施，它其实也是电脑。服务器的配置非常高，服务器级别的CPU动辄12核，甚至服务器还能同时安装多块CPU，能直接堆到好几十核：

家用级CPU一般是AMD的锐龙系列和Intel的酷睿系列（比如i3 i5 i7 i9），而服务器CPU一般是Intel的志强（Xeno）系列，这种CPU的特点就是核心数非常多：

![image-20220630172135408](https://s2.loli.net/2022/06/30/cKlhRZ9Sw1Q4uEX.png)

并且服务器CPU相比家用CPU的功耗更大，发热量非常高。
>但是服务器CPU的频率没有家用级CPU高。一般大型游戏要求的是高频率而不是核心数，再加上服务器CPU功耗高，所以并不适合装在家用电脑中。
在网上买台式机，看到什么“i9级”CPU千万别买，那些是黑心商家把国外服务器上淘汰下来的服务器CPU（洋垃圾）装成电脑来卖，所以会很便宜，同时核心数又能媲美i9。

服务器无论是CPU资源还是内存资源都远超家用电脑，只跑一个小小的Java后端，可能顶多就用服务器5%的硬件资源，非常浪费。

为了解决这种资源利用率只有5%-15%的情况，需要想个办法，把这一台服务器分成多个小服务器使用，每个小服务器只分配一部分的资源。但是由于设计上的问题，电脑只能同时运行一个操作系统，那么怎么办呢？

**虚拟化技术兴起**

此时虚拟化技术就开始兴起了。虚拟化使用*软件来模拟硬件*并创建虚拟计算机系统。这样一来，企业便可以在单台服务器上运行多个虚拟系统，也就是运行多个操作系统和应用。

比如常用的VMware就是一种民用级虚拟化软件：

![image-20220630173915254](https://s2.loli.net/2022/06/30/St3hfELQHNdRZmA.png)

可以使用VMware来创建虚拟机，这些虚拟机实际上都是基于当前系统上的VMware软件来运行的。当然VMware也有服务器专用的虚拟化软件。

有了虚拟化之后，服务器就像这样：

![image-20220630174945749](https://s2.loli.net/2022/06/30/BmnC1xETQM4uRHO.png)

相当于通过虚拟机模拟了很多来电脑出来，这样就可以在划分出来的多台虚拟机上分别安装系统和部署应用程序了。并且可以自由分配硬件资源，合理地使用。

一般在企业中，不同的应用程序可能会被分别部署到各个服务器上，隔离开来，此时使用虚拟机就非常适合。

实际上，在腾讯云、阿里云租的云服务器，都是经过虚拟化技术划分出来的虚拟机而已。

**容器**

容器和虚拟机比较类似，都可以为应用提供封装和隔离，都是软件。

但是容器中的应用运行是**寄托于宿主操作系统**的，实际上依然是在直接使用操作系统的资源（应用程序之间环境依然是隔离的）。

而虚拟机则是完全模拟一台真正的电脑出来，直接就是两台不同的电脑。

![image-20220630181037698](https://s2.loli.net/2022/06/30/31GZSh5DE9Vilet.png)

因此容器相比虚拟机就简单多了，并且启动速度也会快很多，开销小了不少。

不过容器火的根本原因还是它的**集装箱思想**。
比如写一个论坛、电商这类的Java项目，需要用到数据库、消息队列、缓存等这些中间件。因此如果想要将一个服务部署到服务器上，需要提前准备好各种各样的环境：安装MySQL、Redis、RabbitMQ等应用，配置好环境，再将Java应用程序启动。整个流程下来，光是配置环境就要浪费大量的时间。如果是大型的分布式项目，可能要部署很多台机器，项目上个线就要花几天，显然是不科学的。

而容器*可以打包整个环境*。比如将MySQL、Redis等中间件以及Java应用程序一起打包为一个镜像，当需要部署服务时，直接下载镜像运行即可，不需要再进行额外的配置。整个镜像中环境是已经配置好的状态，开箱即用。

![image-20220630182136717](https://s2.loli.net/2022/06/30/NTnU8iSj51CspFw.png)

当下最热门的容器就是Docker了，它的图标上有很多集装箱，每个集装箱代表**环境+应用程序**。
Docker可以将任何应用及其依赖打包为一个轻量级，可移植，自包含的容器，**容器可以运行在几乎所有的操作系统上**。

## 镜像结构

容器的基石是镜像，有了镜像才能创建对应的容器实例。
>可以将镜像看作一个类，容器就是这个镜像（类）创建出来的实例。一个镜像（类）可以同时创建多个容器（实例）

在打包项目时，往往需要一个**基本的操作系统环境**，这样才可以在这个操作系统上安装各种依赖软件，比如数据库、缓存等。这种基本的系统镜像，称为**base镜像**。

一般base镜像就是各个Linux操作系统的发行版，比如Ubuntu，CentOS、Kali等等。

拿Centos镜像为例
查看大小发现只有272M。不像平时使用的完整系统，base镜像的CentOS**省去了内核**：

![image-20220701133111829](https://s2.loli.net/2022/07/01/dvmqAjKHkucbLFh.png)

Linux操作体系由内核空间和用户空间组成。内核空间是整个Linux系统的核心，Linux启动后首先会加`bootfs`文件系统，加载完成后会自动卸载掉，之后会加载用户空间的文件系统，这一层是用户可以进行操作的部分

* bootfs包含了BootLoader和Linux内核，用户不能对这层作任何修改。内核启动之后，bootfs会自动卸载。
* rootfs包含了系统上的常见的目录结构，包括`/dev`、` /proc`、 `/bin`等等以及一些基本的文件和命令。

base镜像底层会**直接使用宿主主机的内核**（这也是缺点所在，如果软件对内核版本有要求的话，那么此时使用Docker就直接寄了），而rootfs则可以在不同的容器中运行多种不同的版本。
所以，base镜像实际上只有CentOS的rootfs，因此很小。其实CentOS镜像里还包含多种基础的软件，而某些操作系统的base镜像甚至精简到不足10M。

```sh
uname -r   //查看内核版本
docker run -it centos   //运行base镜像
ls    //查看根目录下的文件（发现很多命令都没有，比如clear）
```

几乎所有的镜像都是通过**在base镜像的基础上安装和配置需要的软件**构建出来的：

![image-20220701143105247](https://s2.loli.net/2022/07/01/SDwEqz2b7lA9nJa.png)

**分层结构**：每安装一个软件，就在base镜像上叠加一层，这样多个容器都可以将这些不同的层次自由拼装。比如现在好几个容器都需要使用CentOS的base镜像，而上面运行的软件不同，此时分层结构就很有用了，只需要在本地保存一份base镜像，就可以给多个不同的容器拼装使用。

除了这些软件之外，最上层还有一个**可写容器层**
所有的镜像会叠起来组成一个统一的文件系统，如果不同层中存在相同位置的文件，那么上层的会覆盖掉下层的文件，最终看到的是一个叠加之后的文件系统。
需要修改容器中的文件时，实际上并**不会对镜像直接修改**，而是在最顶上的容器层（最上面一般称为容器层，下面都是镜像层）进行修改，不会影响到下面的镜像。这样才能实现镜像给多个容器共享。
各个操作如下：

* 文件读取：Docker从最上层往下依次寻找，找到后则打开文件。
* 文件创建和修改：创建新文件会直接添加到容器层中；修改文件会从上往下依次寻找各个镜像中的文件，如果找到，则将其复制到容器层，再进行修改。
* 删除文件：从上往下依次寻找各个镜像中的文件，一旦找到，并不会直接删除镜像中的文件，而是**在容器层标记这个删除操作**。

也就是说，对整个容器内的文件进行的操作，几乎都是在最上面的容器层进行的，无法干涉到下面镜像层文件，很好地保护了镜像的完整性，实现多个容器共享使用。

## 容器工作机制简述

Docker的整体架构：

![image-20220630184857540](https://s2.loli.net/2022/06/30/PeaxwNQXkiYSlUv.png)

分为三个部分：

* **Docker 客户端**：docker命令都是在客户端上执行的，操作会发送到服务端上处理。
* **Docker 服务端**：服务端是启动容器的主体，一般是作为服务在后台运行，支持远程连接。
* **Registry**：存放Docker镜像的仓库（就像Macen中存放依赖的仓库一样），分为公有和私有仓库。镜像可以从仓库下载到本地存放。

举例

```sh
sudo docker run -d -p 80:80 nginx
```

这个命令输入之后：

1. Docker客户端将操作发送给服务端，告诉服务端要运行nginx这个镜像。
2. Docker服务端先看看本地有没有这个镜像，发现没有。
3. 接着，从公共仓库Docker Hub去查找并下载镜像。
4. 下载完成，镜像保存到本地。
5. Docker服务端加载Nginx镜像，启动容器开始正常运行（容器和其他容器之间，和外部之间，都是隔离的，互不影响）

整个流程中，Docker就像是一搜运输船，镜像就像是集装箱。通过运输船（Docker）将世界各地的货物（镜像）送往目的港口。货物到达港口后，Docker并不关心集装箱（镜像）里面是什么，只需要创建容器开箱即用就可以了。

>不过容器依然是寄托于宿主主机的运行的，所以一般在生产环境下，都是通过虚拟化先创建多台主机，然后再到**各个虚拟机中部署Docker**，这样的话，运维效率就大大提升了。

# 搭建Docker环境

**Windows**：容器主要使用linux内核技术，因此Windows下安装docker可能会有遇到各种问题。

## Ubuntu

选择docker-ce进行安装即可

官方安装文档：https://docs.docker.com/engine/install/ubuntu/

① 卸载已安装的旧版本
旧版本的Docker使用docker、docker.io以及docker-engine的名称，可能还安装了containerd或runc等等。总之，在安装Docker之前，先卸载所有旧版本：
```sh
sudo apt-get remove docker docker-engine docker.io containerd runc
```

② 安装相关前置依赖

```sh
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

③ 添加官方的GPG key：

```sh
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

④ 配置本地软件仓库，将Docker的库添加到apt资源列表中：

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

④ 安装docker engine

```sh
//安装前更新一次apt
sudo apt-get update
//直接安装Docker最新版本。如果上一步没有配置成功，这里会报错找不到相关软件包。
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

⑤ 启动docker，运行hello world查看是否成功
```sh
sudo docker run hello-world
```

配置好后，先退出SSH终端，然后重新连接就可以生效了。

## CentOS

① 卸载已安装的旧版本
旧版本的Docker使用docker、docker.io以及docker-engine的名称，可能还安装了containerd或runc等等。总之，在安装Docker之前，先卸载所有旧版本：
```sh
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate
```

② 更新系统，并安装依赖
```sh
sudo yum update
sudo yum install -y yum-utils
```

③ 添加 Docker 官方 GPG 密钥：
```sh
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

④ 安装docker engine

```sh
//先更新，从刚刚配置的 Docker 软件仓库中获取软件包列表
sudo yum update
//直接安装 Docker 最新版本。如果上一步没有配置成功，这里会报错找不到相关软件包。
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

⑤ 启动docker，运行hello world查看是否成功

```sh
sudo systemctl start docker
sudo docker run hello-world
```
## 方便以后使用


**将当前用户添加到docker用户组中**，不然每次使用docker命令都需要sudo执行，很麻烦：

```sh
sudo usermod -aG docker <用户名>
```

**配置国内镜像仓库地址**
新建/etc/docker/daemon.json文件，输入如下内容：

```bash
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://fsp2sfpr.mirror.aliyuncs.com/"
  ]
}
```

然后重启，配置开机启动
```bash
sudo systemctl restart docker
sudo systemctl enable docker
sudo systemctl enable containerd
```

# 最基本的使用

## 镜像基本操作

**拉取/上传/删除/查看/搜索镜像**

```sh
//从远程仓库拉取镜像(版本默认最新)
docker pull 镜像:版本

//登录远程仓库
docker login -u 用户名

//将本地镜像上传到远程仓库
docker push 镜像:版本

//删除镜像
docker rmi 镜像

//查看本地有哪些镜像
docker images

//从远程仓库搜索镜像
docker search 镜像

//查看镜像构建历史记录
docker history 镜像
```

**运行镜像：自动创建容器并运行**
```sh
//运行镜像（创建容器并运行）
docker run 镜像

//运行镜像，添加`--name`，可以给容器设置指定名称
docker run --name=lbwnb 镜像

//运行镜像，--rm表示容器在停止后自动删除
docker run --rm 镜像

//-p指定宿主机与docker服务器的端口映射，-d表示后台运行
docker run -p 8080:8080 -d 镜像
```

*注意：有一些镜像仅仅run，不会做任何事，因此直接就关掉了，比如base镜像。因此需要在run的时候给它们些事情做。*

```sh
//`-i`表示在容器上打开一个标准的输入接口，
//`-t`表示分配一个伪tty设备，可以支持终端登录
//一般这两个是一起使用，否则base容器启动后就自动停止了。
docker run -it centos
```

## 容器基本操作

**手动启动/终止/暂停/恢复/重启/删除容器**
```sh
//只想创建容器而不想马上运行，可以使用create命令：
docker create 镜像名

//手动开启处于停止状态的容器（容器ID比较长，可以只写部分，只要保证能唯一识别到即可）
docker start <容器名称/容器ID>

//终止正在运行的容器（容器完成善后工作后，才会终止）
docker stop <容器名称/容器ID>

//立即终止正在运行的容器
docker kill <容器名称/容器ID>

//暂停运行指定的容器
docker pause <容器名称/容器ID>

//恢复运行指定的容器
docker unpause <容器名称/容器ID>

//重启容器
docker restart <容器名称/容器ID>

//容器处于非运行状态时才可以删除：
docker rm <容器名称/容器ID>
```

**后端项目运行时（比如SpringBoot项目），几个常用的控制操作**
```sh
//打印日志信息
docker logs 容器名
//持续打印日志信息
docker logs -f 容器名
//容器运行时，进入容器内的终端
docker attach 容器ID/名称
//在容器中创建一个新的终端
docker exec -it 容器名 bash
//在容器中执行一条命令（会在容器创建一个新的终端来执行命令）
docker exec -it 容器名 命令
```

*如果想要退出容器内的终端，需要先按Ctrl+P再按Ctrl+Q来退出。不能直接使用Ctrl+C来终止，这样会直接终止掉Docker中运行的Java程序的*

**查看容器**
```sh
//查看所有的容器及它们的状态（若无-a，只能看到正在运行的容器）
docker ps -a
```

![image-20220701125951980](https://s2.loli.net/2022/07/01/qOblnhr5CJiIBG6.png)

## 构建镜像

### 使用Dokcerfile(推荐)

>例子：创建一个带Java环境的Ubuntu系统镜像。

① 新建名为`Dockerfile`的文件：

```sh
touch Dockerfile
```

② 编辑`Dockerfile`：编写指令来告诉Docker镜像的相关信息

* 首先用`FROM`选择当前镜像的base镜像（必须以这个指令开始。如果不需要任何基础镜像的话，直接使用`scratch`表示从零开始构建）
* base镜像设定后，使用`RUN`指令在容器中运行命令安装Java环境（每条指令执行之后，都会生成一个新的镜像层）

```dockerfile
FROM <基础镜像>
RUN apt update
RUN apt install -y openjdk-8-jdk
```

③ 只需要完成一次构建即可

执行后，Docker会在构建目录中寻找Dockerfile文件，然后开始依次执行Dockerfile中的指令

```sh
docker build -t <镜像名称> <构建目录>
```

构建过程的每一步都非常清晰地列出来了。
并且Docker镜像构建有缓存机制，就算现在中途退出了，然后重新进行构建，也会直接将之前已经构建好的每一层镜像，直接拿来用。除非修改了Dockerfile文件重新构建，只要某一层发生变化其上层的构建缓存都会失效，当然包括`pull`时也会有类似的机制

![image-20220701155443170](https://s2.loli.net/2022/07/01/g6RFwA5t4EsdvnY.png)

④ 成功安装，会出现在本地

![image-20220701155847721](https://s2.loli.net/2022/07/01/95ueUgyaTcrz6Mi.png)

***
使用`history`命令来查看构建历史：

![image-20220701160128689](https://s2.loli.net/2022/07/01/GYyHFcjSKJwvWi6.png)

可以看到最上面两层是通过使用apt命令生成的内容，就直接作为当前镜像中的两层镜像，每层镜像都有一个自己的ID，不同的镜像大小也不一样。

如果遇到镜像ID为missing的一般是从Docker Hub中下载的镜像会有这个问题，但是没有太大影响。
***

### 使用commit(不推荐)

>这种方式不会记录镜像构建的过程，使用者并不知道镜像是如何构建出来的，对里面一无所知。并且这样去构建效率很低，如果要同时构建多种操作系统的镜像，就要要一个个去敲。

① 按需求给容器配置好环境

② 使用`commit`命令将容器保存为新的镜像

```sh
docker commit 容器名称/ID 新的镜像名称
```

![image-20220701152302171](https://s2.loli.net/2022/07/01/sbWLlEoMj2ZPcUV.png)

![image-20220701152418060](https://s2.loli.net/2022/07/01/3q4juA8vOJew9W6.png)

***
使用`history`命令来查看构建历史：

![image-20220701160406891](https://s2.loli.net/2022/07/01/qWUeSF3aKrvwJ8p.png)

发现构建过程根本看不到
***

## 发布镜像到远程仓库

远程仓库Docker Hub：https://hub.docker.com/repositories

① 创建仓库，填写信息：

![image-20220701164609666](https://s2.loli.net/2022/07/01/3T8xJLgER4cWuQq.png)

![image-20220701164939268](https://s2.loli.net/2022/07/01/SkCKJmU6Rw2lfzP.png)

创建完成后，就有了一个公共的镜像仓库，可以将本地的镜像上传到这里。

② 使用`tag`命令来重新打标签，将镜像名称修改得规范一点

这里将版本改成1.0版本，不用默认的latest了
修改完成后，会创建一个新的本地镜像，名称自定义的那个

```sh
docker tag ubuntu-java-file:latest 用户名/仓库名称:版本
```

![image-20220701165231001](https://s2.loli.net/2022/07/01/chAPS2DFW5q7GkE.png)

③ 本地登录

```sh
docker login -u 用户名
```
![image-20220701165446859](https://s2.loli.net/2022/07/01/T3YC4pfaLEo85Oz.png)

④ 登录成功后，上传（上传之后的镜像是被压缩过的）

```sh
docker push 镜像
```

![image-20220701165744647](https://s2.loli.net/2022/07/01/CXoBhpZUl79aDRQ.png)

上传完成后，打开仓库，可以看到已经有一个1.0版本了：

![image-20220701165920060](https://s2.loli.net/2022/07/01/3UD9y8frEIX1JY6.png)

![image-20220701170053250](https://s2.loli.net/2022/07/01/9sVSjcGCo5mTu61.png)

***
测试

先把本地的对应镜像删掉

`search`命令搜索上传的镜像：

```sh
docker search nagocoler/ubuntu-java
```

![image-20220701170253126](https://s2.loli.net/2022/07/01/SIUpBOzN5vsiydn.png)

pull命令将其下载下来：

```sh
docker pull nagocoler/ubuntu-java:1.0
```

![image-20220701171148334](https://s2.loli.net/2022/07/01/uXBk3WPsDM4aZKo.png)

运行试试看：

![image-20220701171253440](https://s2.loli.net/2022/07/01/RJVdstMnxjSYFoW.png)

***

## 用IDEA构建SpringBoot程序镜像

>需求：使用Docker快速地将SpringBoot项目部署到安装了Docker的服务器上，我们就可以将其打包为一个Docker镜像。

① 配置在云服务器/虚拟机中的Docker，开启远程客户端访问

```sh
sudo vim /etc/systemd/system/multi-user.target.wants/docker.service 
```

打开后，添加高亮部分：

![image-20220701202846707](https://s2.loli.net/2022/07/01/OVMDGqiYWU9E7fA.png)

重启Docker服务，如果是云服务器，记得开启2375 TCP连接端口：

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker.service 
```

② 运行maven的package打包命令，生成项目jar包

③ 项目的resource目录下创建Dockerfile

* 基于ubuntu构建一个带Java环境的系统镜像
* COPY命令将jar包拷贝到镜像中。
  * 第一个参数：要拷贝的本地文件
  * 第二个参数：存放在Docker镜像中的文件位置。这里的`app.jar`表示保存在默认路径。
* CMD命令可以设定容器启动后执行的命令：这里指定启动后运行Java程序
* EXPOSE可以指定容器需要暴露的端口，这里暂时不使用，

```dockerfile
FROM ubuntu
RUN apt update && apt install -y openjdk-8-jdk
COPY target/DockerTest-0.0.1-SNAPSHOT.jar app.jar
CMD java -jar app.jar
# EXPOSE 8080
```

④ 配置镜像标记（自定义镜像名称）

![image-20220701204955570](https://s2.loli.net/2022/07/01/edPVg4oyrDiqmk6.png)

![image-20220701205053642](https://s2.loli.net/2022/07/01/1QrHVB4zC9iFTG7.png)

⑤ IDEA连接Docker服务器进行构建

IDEA自带Docker插件，直接点击左上角的运行按钮，选择**为Dockerfile构建镜像**

![image-20220701203741495](https://s2.loli.net/2022/07/01/xB5vEw1QHojWZ8p.png)

![image-20220701202537650](https://s2.loli.net/2022/07/01/FAcME5yxZPD1aoz.png)

在引擎API URL处填写Docker服务器的IP地址：

```
tcp://IP:2375
```

![image-20220701203318098](https://s2.loli.net/2022/07/01/bDn3vHFw1XYdusU.png)

⑥ 连接成功后，就可以在Docker服务器上进行构建

![image-20220701203518930](https://s2.loli.net/2022/07/01/nPFSa4Wcep31jXG.png)

可以看到，Docker服务器上已经有了刚刚构建好的镜像：

![image-20220701205350004](https://s2.loli.net/2022/07/01/6JKXLHEz25QGvMk.png)

***
**测试**

配置完成后，重新构建：

![image-20220701210438145](https://s2.loli.net/2022/07/01/NgCLJbRQc1lMqna.png)

启动镜像，可以直接在IDEA中启动：

![image-20220701210845768](https://s2.loli.net/2022/07/01/t2MV3Tu6IcrK8Dl.png)

![image-20220701210908997](https://s2.loli.net/2022/07/01/JqajY8EdVbGNhiF.png)

启动后可以在右侧看到容器启动的日志信息：

![image-20220701210946261](https://s2.loli.net/2022/07/01/jreyMHzcX8LTh3k.png)

但是现在启动，并不能访问到：这是因为容器内部的网络和外部网络是隔离的，如果想要访问容器内的服务器，需要将对应端口绑定到宿主机上，让宿主主机也开启这个端口，这样才能连接到容器内：

```sh
docker run -p 8080:8080 -d springboot-test:1.0
```

这里`-p`表示端口绑定，将Docker容器内的端口绑定到宿主机的端口上，这样就可以通过宿主的8080端口访问到容器的8080端口了。`-d`参数表示后台运行。

当然直接在IDEA中配置也是可以的：

![image-20220701211536598](https://s2.loli.net/2022/07/01/dXQlEBIDzU6YTLG.png)

配置好后，点击重新创建容器：

![image-20220701211701640](https://s2.loli.net/2022/07/01/6G7hbmW81uBsKFc.png)

重新运行后，就可以成功访问到容器中运行的SpringBoot项目了

![image-20220701211753962](https://s2.loli.net/2022/07/01/7xNrfWcvC58hQ4q.png)
***

## 用IDEA发布镜像到远程仓库

>需求：将刚才构建好的镜像推送到Docker Hub


创建新的公开仓库：

![image-20220701212330425](https://s2.loli.net/2022/07/01/oTXBtlPV7j3C6a9.png)

直接点击

![image-20220701212458851](https://s2.loli.net/2022/07/01/91tKnXDWaeFqcrx.png)

配置Docker Hub相关信息：

![image-20220701212637581](https://s2.loli.net/2022/07/01/tMcD2kzNwW9J7d3.png)

![image-20220701212731276](https://s2.loli.net/2022/07/01/kgTlz3m61ZrHx5s.png)

直接推送即可，等待推送完成。

![image-20220701212902977](https://s2.loli.net/2022/07/01/H5UfWXC2nKVeray.png)

远程仓库中已经出现了刚刚上传的镜像，IDEA中也可以同步看到：

![image-20220701213026214](https://s2.loli.net/2022/07/01/mgRKV2SWb9YxBGr.png)

# 容器网络管理

## 操作汇总

一般的，base镜像都只保留最精简的东西，啥软件都没有，需要手动安装网络管理的软件。
这里用ubuntu镜像为例，安装网络管理软件后打包成新镜像

```sh
docker run -it ubuntu
apt update
apt install net-tools iputils-ping curl
docker commit lucid_sammet ubuntu-net
```

**查询主机上的网络类型**
```sh
//查看主机上的网络类型（默认是none，bridge，host）
docker network ls
//查看当前网络
ifconfig
```

**启动镜像时指定网络类型**
```sh
//自带的有none，bridge，host。也可以用自定义的网络
docker run -it --network=网络名 镜像
//不指定，默认使用的是名为bridge的桥接网络
docker run -it 镜像
//将网络指定为另一个容器的网络，实现两个容器网络共享
docker run -it --name=test01 --network=container:test02 ubuntu-net
```

**查看网络配置信息**
```sh
docker network inspect 网络名
```

**创建自定义网络**
```sh
//三种网络驱动：bridge、overlay、macvlan
docker network create --driver 网络驱动 网络名
```

**当前容器连接到另一个容器所属的网络下**
```sh
docker network connect 网络名 容器ID/名称
```


## 容器网络类型

Docker在安装后，会在主机上创建三个网络，使用`network ls`命令来查看：

```sh
docker network ls
```

![image-20220702161742741](https://s2.loli.net/2022/07/02/7KEumyqriRY2QU5.png)

可以看到默认情况下有`bridge`、`host`、`none`这三种网络类型（有点像虚拟机的网络配置，也是分桥接、共享网络之类的），

* **none网络**：这个网络除了有一个本地环回网络之外，就没有其他的网络了。使用none网络的容器是无法连接到互联网的，“真”单机运行，没人能访问进去，绝对的安全，存点密码这些还是不错的。

* **bridge网络**：容器默认使用的网络类型，即桥接网络。这是应用最广泛的网络类型：

  在宿主主机上查看网络信息，会发现有一个名为docker0的网络设备，这个网络设备是Docker安装时自动创建的虚拟设备。

  ![image-20220702172102410](https://s2.loli.net/2022/07/02/jDKSIriXec96uhy.png)

  默认创建的容器内部的情况：

  ```sh
  docker run -it ubuntu-net
  ```

  ![image-20220702172532004](https://s2.loli.net/2022/07/02/5JdimQWMaCx7hy2.png)

  可以看到容器的网络接口地址为172.17.0.2，实际上这是Docker创建的虚拟网络。就像容器单独插了一根虚拟的网线，连接到Docker创建的虚拟网络上，而docker0网络实际上作为一个桥接的角色，一头是自己的虚拟子网，另一头是宿主主机的网络。

  网络拓扑类似于下面这样：

  ![image-20220702173005750](https://s2.loli.net/2022/07/02/xCKMIBwjq7gWOko.png)

  通过添加这样的网桥，就可以对容器的网络进行管理和控制。
  使用`network inspect`命令来查看docker0网桥的配置信息：

  ```sh
  docker network inspect bridge
  ```

  ![image-20220702173431530](https://s2.loli.net/2022/07/02/86XdZUejEuk1P3i.png)

  这里的配置的子网是172.17.0.0，子网掩码是255.255.0.0，网关是172.17.0.1，也就是docker0这个虚拟网络设备。所以上面创建的容器就是这个子网内分配的地址172.17.0.2了。

* **host网络**：容器连接到此网络后，会共享宿主主机的网络，网络配置也是完全一样的：

  ```sh
  docker run -it --network=host ubuntu-net
  ```

  网络列表和宿主主机的列表是一样的，连hostname都是和外面一模一样的：

  ![image-20220702170754656](https://s2.loli.net/2022/07/02/cRAQtIxV4D9byCu.png)

  只要宿主主机能连接到互联网，容器内部也可以
  这样直接使用宿主的网络，传输性能基本没有什么折损，而且可以直接开放端口等，不需要进行任何的桥接：

  比如安装Nginx之后直接就可以访问了，不需要开放什么端口：

  ```sh
   apt install -y systemctl nginx
   systemctl start nginx
  ```

  相比桥接网络就方便得多。

## 用户自定义网络

除了以上三种网络，还可以自定义网络，让容器连接到这个网络。

Docker默认提供三种网络驱动：`bridge`、`overlay`、`macvlan`，不同的驱动对应着不同的网络设备驱动，实现的功能也不一样。
比如bridge类型的，其实就和前面提到的桥接网络是一样的。

**例子**

创建一个名称为test的桥接网络

```sh
docker network create --driver bridge test
```

![image-20220702180837819](https://s2.loli.net/2022/07/02/piCtK8kdRALHSIu.png)

可以看到新增了一个网络设备，这个就是负责容器网络的网关，和之前的docker0是一样的：
```sh
docker network inspect test
```

![image-20220702181150667](https://s2.loli.net/2022/07/02/uLwAD4YC3UFXQt7.png)

创建一个使用此网络的新容器
成功得到分配的IP地址，是在这个网络内的

```sh
 docker run -it --network=test ubuntu-net
```

![image-20220702181252137](https://s2.loli.net/2022/07/02/Iy2BwDoZsLMO8gJ.png)

再创建一个不同网络的容器：

![image-20220702181808792](https://s2.loli.net/2022/07/02/b14dflKGMunULQI.png)

可以看到不同的网络是相互隔离的，无法进行通信。

可以将此容器连接到另一个容器所属的网络下：

```sh
docker network connect test 容器ID/名称
```

![image-20220702182050204](https://s2.loli.net/2022/07/02/WzvhI63ydfeJStA.png)

这样就连接了一个新的网络：

![image-20220702182146049](https://s2.loli.net/2022/07/02/lxqrz36sVUjNdI4.png)

可以看到容器中新增了一个网络设备连接到自己定义的网络中，现在这两个容器在同一个网络下，就可以相互ping了：
![image-20220702182310008](https://s2.loli.net/2022/07/02/WBlC9PheETO64xq.png)

## 容器间网络

**只要两个容器处于同一个网络下，即可直接通过容器的IP地址在容器间进行通信**

创建两个ubuntu容器：

```sh
docker run -it ubuntu-net
```

先获取其中一个容器的网络信息：

![image-20220702175353454](https://s2.loli.net/2022/07/02/yTEcg4l2kASBnQu.png)

可以直接在另一个容器中ping通这个容器，因为这两个容器都是使用的bridge网络，在同一个子网中。

**也可以通过容器名进行通信（只会在自定义的网络下生效，默认网络不生效）**

大部分情况下，容器部署之后的IP地址是自动分配的（可以用`--ip`手动指定，但还是不方便），无法提前得知IP地址，用IP通信的做法不灵活。

这时可以借助Docker提供的DNS服务器。

只需要在容器启动时给个名字，直接访问这个名称，会被解析为对应容器的IP地址。

```sh
docker run -it --name=test01 --network=test ubuntu-net
docker run -it --name=test02 --network=test ubuntu-net
```

直接ping对方的名字就可以了：

![image-20220702192457354](https://s2.loli.net/2022/07/02/lKCFY6ec17N4b5y.png)

**两个容器共享同一个网络**
这种情况，两个容器共同使用一个IP地址

```sh
docker run -it --name=test01 --network=container:test02 ubuntu-net
```

这里将网络指定为另一个容器的网络，这样两个容器使用的就是同一个网络了：

![image-20220702200711351](https://s2.loli.net/2022/07/02/Wb6jODxFP3r1mE7.png)

可以看到两个容器的IP地址和网卡的Mac地址是完全一样的，此时在容器中访问localhost，既是自己也是别人。

在容器1中，安装Nginx，然后再容器2中访问：

```sh
 apt install -y systemctl nginx
 systemctl start nginx
```

![image-20220702201348722](https://s2.loli.net/2022/07/02/WTn9OMYmLZJXtBz.png)

成功访问到另一个容器中的Nginx服务器。

## 容器外部网络

容器如何与外网通信？

在默认的三种的网络下，只有共享模式和桥接模式可以连接到外网。共享模式实际上就是直接使用宿主主机的网络设备连接到互联网。

这里主要来看桥接模式：**NAT+端口映射（这个要自己配）**

桥接模式会创建一个单独的虚拟网络，让容器在这个虚拟网络中，然后通过桥接器与外界相连。与外网通信时，数据包从容器内部的网络到达宿主主机，然后再发送到外网。

整个过程的关键是依靠NAT进行IP地址转换，再利用宿主主机的IP地址发送数据包。

![image-20220702232449520](https://s2.loli.net/2022/07/02/ktEA5O9BrmxXbPz.png)

单纯依靠NAT的话，只能容器主动与外界联系，外界无法主动联系容器。而容器可能会部署一些服务，需要外界来主动连接。

解决方法是，在启动容器时配置端口映射，比如：

```sh
docker run -d -p 80:80 nginx
```

`-p`参数进行端口映射配置，将容器需要对外提供服务的端口映射到宿主主机的端口上。这样，当外部访问到宿主主机的对应端口时，就会直接转发给容器内映射的端口。规则为`宿主端口:容器端口`。

![image-20220702233420287](https://s2.loli.net/2022/07/02/WQzEVTwePNaHYgG.png)

一旦监听到宿主主机的80端口收到了数据包，那么会直接转发给对应容器的对应端口。

配置端口映射之后，才可以从外部正常访问到容器内的服务。

# 容器存储管理

## 操作汇总

**将宿主机指定目录/文件挂载到容器指定位置**
```sh
//如果宿主机没有指定目录路径，Docker会自动创建一个目录，
//并且将容器中对应路径下的内容拷贝到这个自动创建的目录中，
//然后挂载到容器
docker run -it -v 宿主机指定目录路径:挂载路径 镜像名

//--volumes-from继承一个容器的数据卷
docker run -p 80:80 --volumes-from=data_test -d nginx
```

**删除数据卷**
```sh
//查看现存的数据卷
docker volume ls
//删除指定数据卷
docker volume rm 数据卷名
```

## 容器持久化存储

在容器中创建和修改的文件，实际上是被容器的分层机制保存在最顶层的容器层进行操作的。这个CopyOnWrite特性，是为了保护下面每一层镜像不被修改。

但是这会导致容器在销毁时数据丢失，重新创建新容器，直接回到梦开始的地方。

如果希望对容器内的某些文件进行持久化存储，就要用到**数据卷（Data Volume）**

**将文件保存到宿主主机上，这样就算容器销毁，文件依然在宿主主机上保留，下次创建容器时，从宿主主机读取对应的文件即可**

**具体做法是，将宿主主机上的文件直接挂载到容器中，容器直接访问宿主主机上的文件**

***
**实验**

用到的镜像：

```sh
docker run -it ubuntu
apt update && apt install -y vim
docker commit
```

用户目录下创建一个新的`test`目录，在里面创建一个文件，写点内容：

```sh
mkdir test
vim test/hello.txt
```

将宿主主机上的目录或文件挂载到容器的某个目录上：

```sh
docker run -it -v ~/test:/root/test ubuntu-volume
```

`-v`用于指定文件挂载。这里将刚刚创建好的test目录挂载到容器的/root/test路径上。

![image-20220703105256049](https://s2.loli.net/2022/07/03/ztEJDC4PTVAyZF2.png)

上图可以看到，直接在容器中就能访问宿主主机上的文件了。

**对挂载目录进行编辑，编辑的是宿主主机的数据。**
比如下面在挂载目录中创建一个test.txt文件

```sh
vim /root/test/test.txt  
```

![image-20220703105626105](https://s2.loli.net/2022/07/03/YqUHkJiTG3Q9pAM.png)

在宿主主机的对应目录下，可以直接访问到刚刚创建好的文件。

容器销毁后，挂载的数据依然保留：

![image-20220703105847329](https://s2.loli.net/2022/07/03/B5M6Wy8AxIoqJtC.png)

***

**例子：nginx代理前端项目**

直接将前端页面保存到宿主主机上，然后通过挂载的形式让容器中的Nginx访问。
就算之后Nginx镜像有升级，需要重新创建，也不会影响到前端页面。


```sh
//将前端模板上传到服务器
scp Downloads/moban5676.zip 192.168.10.10:~/
//解压
unzip moban5676.zip
//启动nginx容器，将解压出来的目录挂载到容器中Nginx的默认站点目录`/usr/share/nginx/html/`
docker run -it -v ~/moban5676:/usr/share/nginx/html/ -p 80:80 -d nginx
//重启nginx服务
systemctl start nginx
```

将解压出来的目录，挂载到容器中Nginx的默认站点目录`/usr/share/nginx/html/`（由于挂载后位于顶层，会替代镜像层原有的文件），Nginx直接代理了存放在宿主主机上的前端页面。别忘了把端口映射到宿主主机上。

通过浏览器访问发现代理成功：

![image-20220703111937254](https://s2.loli.net/2022/07/03/YtgXWizh765qFxr.png)

***

**不指定宿主机上的目录进行挂载**

使用`-v`参数时不指定宿主主机上的目录进行挂载的话，Docker会自动创建一个目录，并且会将容器中对应路径下的内容拷贝到这个自动创建的目录中，然后挂载到容器。这种就是由Docker管理的数据卷（docker managed volume）

```sh
docker run -it -v /root/abc ubuntu-volume
```
这里仅仅指定了挂载路径，没有指定宿主主机的对应目录：

![image-20220703112702067](https://s2.loli.net/2022/07/03/fXCl7IRqKBvYwxj.png)

创建后可以看到`root`目录下有一个新的`abc`目录

使用`inspect`命令查找它在宿主主机的位置

```sh
docker inspect bold_banzai 
```

![image-20220703113507320](https://s2.loli.net/2022/07/03/zFotAfeBpcRjKWN.png)

可以看到Sorce指向的是`/var/lib`中的某个目录。
进入这个目录来创建一个新的文件（记得先提升权限）

![image-20220703114333446](https://s2.loli.net/2022/07/03/2bfokiMTmdGZcUE.png)

![image-20220703114429831](https://s2.loli.net/2022/07/03/yi1hSPC3bAndMXm.png)

和之前一样，也是可以在容器中看到的，容器销毁后，数据依然保留。

***

**不挂载目录**

为了方便，有时并不需要直接挂载一个目录上去。
可以用`cp`命令在宿主主机和容器间传递一些文件

![image-20220703115648195](https://s2.loli.net/2022/07/03/uw7S5PobAUWBtCI.png)

## 容器数据共享

**两个容器挂载同一个目录实现数据共享**

继续使用**挂载的思路**：在宿主主机创建一个公共的目录，让需要实现共享的容器，都挂载这个公共目录：

```sh
docker run -it -v ~/test:/root/test ubuntu-volume
```

![image-20220703141840532](https://s2.loli.net/2022/07/03/soxdKyY4MIXBOin.png)

两个容器挂载宿主主机上的同一块区域，这两个容器都能访问到这里面的数据。

也可以将容器B挂载的目录指定使用容器挂载A的目录
这里使用`--volumes-from`指定另一个容器，这种用于给其他容器提供数据卷的容器，称为**数据卷容器**

```sh
docker run -it -v ~/test:/root/test --name=data_test ubuntu-volume
docker run -it --volumes-from data_test ubuntu-volume
```

![image-20220703142849845](https://s2.loli.net/2022/07/03/Uu4CjSZifv1Oyr7.png)

数据卷容器中挂载的内容，在当前容器中也是存在的。就算此时数据卷容器被删除，也不会影响到这边。因为这边相当于是继承了数据卷容器提供的数据卷，本质上还是让两个容器挂载了同样的目录实现数据共享。

***
**将数据放在容器中进行共享**

通过上面的方式实际上并不方便。某些时候仅仅是希望容器之间共享，而不希望有宿主主机这个角色直接参与到共享之中。
**可以将数据完全放入到容器中，直接将容器中打包好的数据分享给其他容器**。本质上依然是一个Docker管理的数据卷，虽然还是没有完全脱离主机，但是移植性高得多。

Dockerfile：

```dockerfile
FROM ubuntu
//复制文件到容器中，并自动解压
ADD moban5676.tar.gz /usr/share/nginx/html/
//VOLUME指令和`-v`参数一样，会创建一个挂载点在容器中
VOLUME /usr/share/nginx/html/
```

```sh
cd test
tar -zcvf moban5676.tar.gz *
mv moban5676.tar.gz ..
cd ..
```

直接构建：

```sh
docker build -t data .
```

![image-20220703153109650](https://s2.loli.net/2022/07/03/M7jxBUsApKtgzku.png)

现在运行一个容器：

![image-20220703153343461](https://s2.loli.net/2022/07/03/SUg32jlwMcY7Btp.png)

可以看到所有的文件都自动解压出来了（中文文件名称，不过无关紧要）

退出容器，可以看到数据卷列表中新增了这个容器需要使用的：

![image-20220703153514730](https://s2.loli.net/2022/07/03/m6VCIbXyMxt3ilT.png)

![image-20220703153542739](https://s2.loli.net/2022/07/03/KyLUic5r6oW4HDx.png)

这个位置实际上就是数据存放在当前主机上的位置了，不过是由Docker进行管理而不是自定义的。
现在可以创建一个新的容器直接继承了：

```sh
docker run -p 80:80 --volumes-from=data_test -d nginx
```

访问一下Nginx服务器，可以看到成功代理：

![image-20220703111937254](https://s2.loli.net/2022/07/03/YtgXWizh765qFxr.png)

自此就实现了将数据放在容器中进行共享，不需要刻意去指定宿主主机的挂载点，而是Docker自行管理。这样就算迁移主机依然可以快速部署。


# 容器资源管理

## 物理资源管理

对于一个容器，有时并不希望它占据所有的系统资源，而是希望只分配一部分资源给容器。

### 分配内存

`-m`对容器的物理内存限制，`--memory-swap`对内存和交换分区总和进行限制。
它们默认都是`-1`，没有任何限制（如果仅指定`-m`参数，那么交换内存的限制与其保持一致，内存+交换等于`-m`的两倍大小）默认情况下跟宿主主机一样，都是2G内存。

```sh
docker run -m 内存限制 --memory-swap=内存和交换分区总共的内存限制 镜像名称
```

***
**实验**

现在将容器的内存限制到100M：物理内存50M，交换内存50M

```sh
docker run -it -m 50M --memory-swap=100M nagocoler/springboot-test:1.0
```

发现因为内存不足无法启动

![image-20220702104653971](https://s2.loli.net/2022/07/02/MrBWZKIzgxE94Ck.png)

### 对CPU资源进行限额

默认情况下，所有容器平等地使用CPU资源。
* `-c`选项（或全名`--cpu-share`）调整容器的CPU权重（默认为1024），实现按需分配资源
  比如说，两个容器的CPU权重为2比1（注意：多个容器时才会生效）。
当CPU资源紧张时，会按照此权重来分配资源；如果CPU资源不紧张，依然有机会使用到全部的CPU资源。
* `--cpuset-cpus`限制容器在指定的CPU上运行
* `--cpus`限制容器使用的CPU资源数
* 还有更精细的`--cpu-period `和`--cpu-quota`，去阅读文档


***
**实验`-c`选项**

使用一个压力测试工具来进行验证

```sh
docker run -c 1024 --name=cpu1024 -it ubuntu
docker run -c 512 --name=cpu512 -it ubuntu
```

分别进入容器安装`stress`压力测试工具：

```sh
apt update && apt install -y stress
```

分别在两个容器中启动压力测试工具，产生4个进程不断计算随机数的平方根：

```sh
stress -c 4
```

进入top查看CPU状态

![image-20220702114126128](https://s2.loli.net/2022/07/02/3dHkMWnq1ZxCyKm.png)

可以看到，权重高的容器分配到了更多的CPU资源，而权重低的容器中，只分配到一半的CPU资源。
***

**实验`--cpuset-cpus`**

比如现在的宿主机是2核的CPU，可以分0和1这两个CPU给Docker使用。限制后，只会使用CPU 1的资源了

```sh
docker run -it --cpuset-cpus=1 ubuntu
```

![image-20220702115538699](https://s2.loli.net/2022/07/02/erovkRBi7hSOuAt.png)

可以看到，4个进程只各自使用了25%的CPU，加在一起就是100%，也就是只能占满一个CPU的使用率。

如果要分配多个CPU，则使用逗号隔开：

```sh
docker run -it --cpuset-cpus=0,1 ubuntu
```

***

**实验`--cpus`**

```sh
docker run -it --cpus=1 ubuntu
```

![image-20220702120329140](https://s2.loli.net/2022/07/02/pUGCjlsQbEM2Ika.png)

限制为1后，只能使用一个CPU提供的资源

### 限制磁盘IO读写性能

**通过`--device-read/write-bps`和`--device-read/write-iops`参数磁盘读写性能限制**

* bps：每秒读写的数据量。
* iops：每秒IO的次数。

***
**实验**

使用`dd`命令来测试磁盘读写速度：

```sh
dd if=/dev/zero of=/tmp/1G bs=4k count=256000 oflag=direct
```

不用等待跑完，中途Ctrl+C结束就行：

![image-20220702121839871](https://s2.loli.net/2022/07/02/1y3O2qbaMsxDFUJ.png)

可以看到当前的读写速度为86.4 MB/s。

这里使用BPS作为限制条件：

```sh
docker run -it --device-write-bps=/dev/sda:10MB ubuntu
```

因为容器的文件系统是在`/dev/sda`上的，所以填`/dev/sda:10MB`来限制对/dev/sda的写入速度只有10MB/s

![image-20220702122557288](https://s2.loli.net/2022/07/02/EczxDAmUCvlwT5u.png)

可以看到现在的速度就只有10MB左右了。

## 容器监控

### status/top命令

`stats`命令可以实时对容器的各项状态进行监控，包括内存使用、内存分配限制、CPU占用、网络I/O、磁盘I/O等信息

```sh
docker stats
```

`top`命令查看容器中的进程
可以携带一些参数，具体的参数与Linux中`ps`命令参数一致

```sh
docker top 容器ID/名称
```

### Portainer可视化

除了敲命令之外，可以选择部署一个Docker网页管理面板应用进行监控和管理，比较常用的有：Portainer。

通过Docker镜像的方式去部署Portainer应用程序。
最新版维护的地址为：https://hub.docker.com/r/portainer/portainer-ce
也可以使用非官方的汉化版本：https://hub.docker.com/r/6053537/portainer-ce。

CE为免费的社区版本（这个够了），官方Linux安装教程：https://docs.portainer.io/start/install/server/docker/linux

安装及部署流程：

① 创建一个数据卷供Portainer使用：

```sh
docker volume create portainer_data
```

② 通过官方命令安装启动

注意这里需要开放两个端口，一个是8000端口，还有一个是9443端口

```sh
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

③ 开启成功，可以直接登录后台面板：https://IP:9443/ 

这里需要HTTPS访问，浏览器可能会提示不安全，无视就行：

![image-20220702155637366](https://s2.loli.net/2022/07/02/mukzgvnWZyrxeaM.png)

④ 按照提示进行注册，默认用户名就是admin

⑤ 注册完成后，就可以正常使用了

点击Get Started进入到管理页面

![image-20220702160124676](https://s2.loli.net/2022/07/02/P1JIKaMCl7guYoz.png)

看到目前有一个本地的Docker服务器正在运行
可以点击进入，进行详细地管理

![image-20220702160328972](https://s2.loli.net/2022/07/02/OUTrAEmwsNoSG8Y.png)

# Docker-Compose单机容器编排

对容器进行编排：
比如现在要在一台主机上部署很多种类型的服务，包括数据库、消息队列、SpringBoot应用程序若干，或是想要搭建一个MySQL集群，这时就需要创建多个容器来完成。
同时希望能够一键部署，那么就需要用到容器编排了，让多个容器按照我自定义的编排进行部署。

**更多配置操作，看官方文档**：https://docs.docker.com/get-started/08_using_compose/
