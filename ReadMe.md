







## 1. 安装／升级Docker客户端

推荐安装1.10.0以上版本的Docker客户端，参考文档[docker-ce](https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.58801991pLDtaT)

## 2. 配置镜像加速器

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://1056mbof.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```















## 1、找镜像

[docker hub](http://hub.docker.com/)

```bash
docker pull nginx  #下载最新版

镜像名:版本名（标签）

docker pull redis  #下载最新
docker pull redis:6.2.4

## 下载来的镜像都在本地
docker images  #查看所有镜像

redis = redis:latest

docker rmi 镜像名:版本号/镜像id
```

## 2、启动容器

> 启动nginx应用容器，并映射88端口，测试的访问

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

【docker run  设置项   镜像名  】 镜像启动运行的命令（镜像里面默认有的，一般不会写）

# --name=mynginx 名字叫mynginx
# -d：后台运行
# --restart=always: 开机自启
# -p  88:80  主机的88端口映射到容器的80端口
docker run --name=mynginx   -d  --restart=always -p  88:80   nginx

# 查看正在运行的容器
docker ps
# 查看所有
docker ps -a
# 删除停止的容器
docker rm  容器id/名字
docker rm -f mynginx   #强制删除正在运行中的

#停止容器
docker stop 容器id/名字
#再次启动
docker start 容器id/名字

#设置应用开机自启   但是update不能更新端口映射
docker update 容器id/名字 --restart=always
```

## 3、修改容器内容

> 修改默认的index.html 页面

### 1、进入容器内修改

```bash
# 进入容器内部的系统，修改容器内容
docker exec -it 容器id  /bin/bash
#或者
docker exec -it 容器id  /bin/sh
```

### **2、挂载数据到外部修改**

```bash
#-v 数据挂在
#ro 只读模式
docker run --name=mynginx   \
-d  --restart=always \
-p  88:80 -v /data/html:/usr/share/nginx/html:ro  \
nginx

# 修改页面只需要去 主机的 /data/html
```



## 4、提交改变

> 将自己修改好的镜像提交

```BASH
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

docker commit -a "leifengyang"  -m "首页变化" 341d81f7504f guignginx:v1.0
```

### **1、镜像传输**

```bash
# 将镜像保存成压缩包
docker save -o abc.tar guignginx:v1.0

# scp命令复制文件到远程服务器。指定远程用户名和目录，命令执行后需要再输入密码
scp local_file remote_username@remote_ip:remote_folder 

# 别的机器加载这个镜像
docker load -i abc.tar


# 离线安装
```

## **5、推送远程仓库**

> 推送镜像到docker hub；应用市场

```bash
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```



```bash
# 把旧镜像的名字，改成仓库要求的新版名字
docker tag guignginx:v1.0 leifengyang/guignginx:v1.0

# 登录到docker hub
docker login       


docker logout（推送完成镜像后退出）

# 推送
docker push leifengyang/guignginx:v1.0


# 别的机器下载
docker pull leifengyang/guignginx:v1.0
```

## **6、补充**

```bash
docker logs 容器名/id   排错

docker exec -it 容器id /bin/bash


# docker 经常修改nginx配置文件
docker run -d -p 80:80 \
-v /data/html:/usr/share/nginx/html:ro \
-v /data/conf/nginx.conf:/etc/nginx/nginx.conf \
--name mynginx-02 \
nginx


#把容器指定位置的东西复制出来 
docker cp 5eff66eec7e1:/etc/nginx/nginx.conf  /data/conf/nginx.conf
#把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf  5eff66eec7e1:/etc/nginx/nginx.conf
```

