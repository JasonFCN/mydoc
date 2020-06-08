#### docker安装

```shell

# 卸载旧版本
sudo apt-get remove docker docker-engine docker.io containerd runc
# 配置公钥
	# 国外
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	# 国内
	curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# 配置docker仓库地址
    # 国外仓库
    sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    # 国内仓库
    sudo add-apt-repository \
       "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
       $(lsb_release -cs) \
       stable"

# 安装
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
# 启动docker服务
systemctl start docker
# 查看版本信息
docker version
# 测试是否安装成功
sudo docker run hello-world

# 卸载
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
```

#### 镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://81j7hyho.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### docker常用命令

```shell
docker version		# 显示客户端/服务端版本信息
docker info 		# 显示docker系统信息（如镜像或者容器数量）
docker xx --help	# 命令帮助信息
```

##### 镜像命令

```shell
docker images		# 显示镜像信息 -a:显示所有 -q:只显示id
```

```shell
docker search		# 搜索镜像 --filter=START=3000 搜索stars大于3000的
```

```shell
docker pull			# 下载镜像 [:tag] 指定版本 ，不指定默认为最新latest
```

```shell
docker rmi			# 删除镜像 -f 递归 docker rmi -f $(docker images -aq) 删除所有镜像
```

##### 容器命令

```shell
docker run 			# 运行一个容器 
# eg: docker run --name centos01 -p 8080:8080 -it centos /bin/bash
# --name 	:容器命名
# -p		:开放端口
# -it		:交互运行；进入容器后 exit命令退出
```

```shell
docker ps			# 查看容器列表
# -a	:列出所有
# -q	:只显示容器id
```

```shell
# 退出容器
exit	#直接停止容器并退出
Ctrl + P + Q	# 不停止退出

```

```shell
# 容器删除操作
docker rm 						# 容器删除
docker rm id					# 删除指定id的容器
docker rm id id id				# 删除多个容器
docker rm -f $(docker ps -qa)	# 删除所有容器
docker ps -a -q|xargs docker rm # 删除
```

```shell
# 容器的启动与停止
docker start id		# 启动容器
docker restart id 	# 重启容器
docker stop id		# 停止正在运行的容器
docker kill id		# 强制停止正在运行的容器
```

##### 其他命令

```shell
# 后台启动后的容器，被停止  原因：容器需要前置应用
docker logs		# 查看日志
-f				# 跟随日志输出
-t 				# 时间戳
--tail number   # 显示记录条数

docker top id 	# 显示进程信息
docker inspect  # 显示容器元数据 
docker exec -it id bashShell 	# 进入正在运行的容器,打开新的终端
docker attach id				# 进入正在运行的容器，自动进入正在运行的终端
docker cp						# 拷贝文件到主机
# eg: docker cp id:/home/test.java /home
```

##### 更多

![docker命令](docker.assets/docker命令.png)

#### 启动es

```shell
# 默认启动方式会占用大量内存
docker run -d --name es01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch
# 配置参数
docker run -d --name es01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch

# 查看容器运行状态
docker stats id
```

#### 启动tomcat

```shell
# 启动一个容器
docker run -d --name tomcat01 -p 8080:8080 tomcat
# 进入容器，拷贝文件
docker exec -it id /bin/bash
cp webapps.dist/* webapps
# 提交到镜像
docker commit -a="cwj" -m="webapps下拷贝文件" 08b tomcat01:1.0
# 查看镜像列表
(base) jason@jason-X6Ti:~/下载$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat01            1.0                 60853b6b74d0        50 seconds ago      652MB
tomcat              latest              1b6b1fe7261e        9 days ago          647MB

```

#### 数据卷技术

##### 文件挂载

容器与主机相互隔离，然而容器中的程序运行产生的数据会随着容器的删除而删除。生产环境是不允许的。

这就使得docker必须实现一种主机与容器间有交互的文件共享技术：**文件挂载**

在运行容器时，通过-v来指定主机与容器间的文件挂载方式：

```shell
docker run -d --name tomcat02 -p 8080:8080 -v /home/logs/tomcat:/usr/local/tomcat/logs -it tomcat01:1.0 /bin/bash
```

查看容器详细信息：

```shell
docker inspect dbaa

# 显示信息中:
 "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/logs/tomcat",
                "Destination": "/usr/local/tomcat/logs",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

```

***当任何路径下有文件变化时，对方都能同步相同的变化。***

***当容器停止运行后，对主机挂载的路径文件做更改，也会同步到容器中。***

***当容器删除时，主机下挂载的文件并不会删除。***

##### 作业：mysql路径挂载

```shell
# -e MYSQL_ROOT_PASSWORD=123456 设置ROOT密码
docker run -d --name mysql01 -p 3306:3306 -v /home/jason/mysql/data:/var/lib/mysql -v /home/jason/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

```

##### 数据卷挂载：

```shell
-v name:/etc/tomcat  	# 具名挂载
-v /etc/tomcat			# 匿名挂载
会在/var/lib/dokcer/volumes下生成生成文件hash值/name的文件夹，里面就是我们容器内挂载的文件
-v /etc/tomcat:ro		# 设置容器内文件权限 ro:readonly(容器不能写操作该文件，只能宿主机可以写)； rw:readwrite(默认)
```

#### dockerfile

用来构建镜像的文件

##### 初体验：

1，创建一个文件Dockerfile；

2，编辑Dockerfile，写入：

```shell
FROM ubuntu						# 以ubuntu为基础

VOLUME ["volume01", "vomume02"]	# 挂载数据卷

CMD echo "balabala"				# 输出 。。

CMD /bin/bash					# 进入命令行
```

保存

3，通过build创建镜像

```shell
docker build -f Dockerfile -t imageName .
-f	# 指定dockerfile文件位置
-t  # tag/name
```

eg:

```shell
docker build -f dockerfile -t myubuntu01 .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete 
fc878cd0a91c: Pull complete 
6154df8ff988: Pull complete 
fee5db0ff82f: Pull complete 
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
 ---> 1d622ef86b13
Step 2/3 : CMD echo "hello myubuntu"
 ---> Running in fe867e7ea3fb
Removing intermediate container fe867e7ea3fb
 ---> 0f1fa0c19a23
Step 3/3 : CMD /bin/bash
 ---> Running in 2a082353c247
Removing intermediate container 2a082353c247
 ---> 75ad2dcc7209
Successfully built 75ad2dcc7209
Successfully tagged myubuntu01:latest

docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
myubuntu01          latest              75ad2dcc7209        About a minute ago   73.9MB
```

dockerfile常用命令：

```shell
FROM		# 以谁为基础来构建镜像
MAINTAINER	# 指定维护者信息（姓名+邮箱）
RUN			# 构建过程中需要执行的命令
ADD			# 添加文件
WORKDIR		# 工作目录
VOLUME		# 设置卷
EXPOSE		# 暴露端口
CMD			# 容器启动时要运行的命令 只有最后一个会生效(总是被后面的CMD命令替换)
ENTRYPOINT	# 容器启动时要运行的命令 可追加
COPY		# 类是ADD,拷贝文件
ENV			# 设置环境变量
```





