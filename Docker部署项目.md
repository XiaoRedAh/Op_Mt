传统项目部署：买服务器 -> 装系统 -> 装环境（Java、Nginx、Nodejs）-> 上传项目 -> 编写命令运行项目 -> 服务器重启 -> 重复命令，如果是分布式项目，可能会有很多台服务器，各个不同的服务要在不同的服务器上运行，岂不是要每一个服务器都去装环境、敲命令来运行？

Docker可以把项目打包成一个镜像，比如现在有前端项目和后端项目要分别部署在两台服务器上，那么可以先将其打包为Docker镜像然后上传到DockerHub容器仓库：买服务器 -> Docker系统 -> 从DockerHub拉取镜像然后运行 -> 服务器重启 -> 不需要操作，自动启动

# 本地镜像打包

本地下载一个桌面版的Docker方便操作

## 前端项目打包为镜像

① 前端项目的baseURL改成以后需要部署到的服务器的地址和端口

② 编译前端项目

③ 创建Dockerfile文件
* 为镜像添加nginx环境
* 将编译后的前端项目(dist目录下)，拷贝到服务器的/web目录(这个，目录是自定义的)
* 将编写好的nginx配置文件拷贝到给default.conf，使nginx代理自定义的web目录
```dockerfile
FROM nginx
COPY dist/ /web
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

④ 创建nginx.conf文件
nginx不会自动代理自定义的web目录，需要配置
```nginx
server{
    listen  80;
    server_name localhost;
    location / {
        root    /web
        index   index.html
        try_files $uri
    }
}
```

⑤ cd到前端目录，将前端项目打包成镜像
```bash
docker build -t 镜像名 
```

打开桌面端的docker，发现已经成功生成前端项目的镜像了，run运行即可在localhost访问到它

## 后端项目打包为镜像

# 服务器部署

>把打包好的镜像上传到dockerhub上，服务器去拉取dockerhub上的这个镜像

① cd到前端目录
```bash
docker tag 镜像名 远程地址
docker login
docker push 远程仓库
```

② 服务器上拉取刚推送的镜像

```bash
sudo docker login
sudo docker pull 远程仓库
```

现在，只需要服务器上run这个镜像，用服务器地址和对应端口就能访问到前端项目了

# Portainer可视化部署

```bash
//为了方便，以后不用敲sudo，先切换到root用户
sudo -s
//切换到/root目录下
cd
```

① 下载并解压Portainer汉化包
得到一个public目录
```bash
wget http://code.imnks.com/zip/portainer-ce-public-cn-20221227.zip
unzip portainer-ce-public-cn-20221227.zip
```

② 创建一个目录作为Portainer的数据目录，方便后续迁移
现在/root目录下有一个public目录和一个data目录
```bash
mkdir data
```

③ 启动时将Portainer的public目录挂载到宿主主机刚刚解压出来的目录下

```bash
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /root/data:/data -v /root/public:/public portainer/portainer
```

④ 服务器9000防火墙打开

完成上述操作后，访问服务器公网IP:9000即可进行可视化管理。



# 搭建私有Docker Registry

搭建一个私有的容器仓库，不能随意访问，因此要先创建验证密码才可以

```bash
yum install httpd-tools
```

创建密码
将密码复制到/root/registry/auth/passwd目录下（其他目录也可以）【加密存储】
```bash
htpasswd -Bbn admin 密码 > /root/registry/auth/passwd
```

服务器开放5000端口

直接使用以下命令，然后就可以往自己的私有Docker仓库上传镜像了：
```bash
docker run -d -p 5000:5000 -v /root/registry/auth:/etc/registry/auth -v /root/registry/data:/var/lib/registry -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/etc/registry/auth/passwd" registry
```

Mac/Windows处理HTTP问题：

"insecure-registries":["仓库ip:port"]
Linux：

--insecure-registry ip:port