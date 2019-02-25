# Docker 学习筆記

**[Docker命令大全](http://www.runoob.com/docker/docker-command-manual.html)**
## 安装Docker
CentOS

`#yum install docker-ce`

Ubuntu
`#apt-get install docker-ce`
启动 docker 守护进程
`#systemctl start docker`
脚本安装
```
# curl -sSL https://get.docker.com | sh

国内的主机：
# curl -sSL https://get.daocloud.io/docker | sh
sudo chkconfig docker on
sudo systemctl start docker
sudo usermod -aG docker your-user  //添加权限
```

## Hello Docker
`# docker run -it centos:latest bash  会进入拉取centos镜像运行所在容器中`

## Dockerfile

常用命令
docker run 新建并启动容器
- -d 容器运行在后台，此时不能使用--rm选项
- -i -t 和容器进行交互式操作
- --name 命名容器，没有该参数Docker deamon会生产UUID来标识
- --cidfile 将容器ID输入到指定文件中
- --add-host 添加一行到/etc/hosts
- --mac-address 设置MAC地址
- --dns 覆盖容器DNS设置
- --rm 退出容器时自动清除数据
- -m 调整容器的内存使用
- -c 调整容器的CPU优先级
- -e 设定环境变量
- --expose 运行时暴露端口，不创建和宿主机的映射
- -p 创建映射规则，将一个或者一组端口从容器里绑定到宿主机上,可多次使用
* ip:hostPort:containerPort
* ip::containerPort
* hostPort:containerPort
* containerPort
- -P 将Dockfile中暴露的端口映射动态映射到宿主机
- --link 容器互联 --link name:alias
- -v 创建数据卷挂载到容器，一次run中可多次使用
可覆盖Dockfile参数
- CMD
- ENTRYPOINT
- EXPOSE
- ENV
- VOLUME
- USER
- WORKDIR
* docker stop 停止运行中容器
* docker stop $(docker ps -qa) 停止所有运行中的容器
* docker restart 重启容器
* docker ps -a 查看所有容器
* docker rm 移除处于终止状态的容器
* `docker rm $(docker ps -qa)` 移除处于终止状态的容器
* docker logs 从容器中去日志
* docker diff 列出容器中被改变的文件或者目录
* docker top 显示运行容器的进程信息
* docker cp 从容器中拷贝文件或者目录到本地
* docker inspect 查看容器详细信息

## 使用Docker Compose编排容器
```
# curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
```
