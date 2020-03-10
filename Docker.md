[TOC]
# Docker
## 安装
1. 安装依赖  
docker依赖于系统的一些必要的工具，可以提前安装  
`yum install -y yum-utils device-mapper-persistent-data lvm2`
2. 添加软件源  
`yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
3. 安装docker-ce  
`yum -y install docker-ce`
4. docker镜像加速  
修改`/etc/docker/daemon.json`  
```
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
```
5. 启动服务  
`systemctl start docker`

## 操作命令
### 镜像
- 搜索镜像
`docker search centos`
- 拉取镜像
`docker pull centos`
- 查看本地镜像
`docker images`
- 删除本地镜像
`docker rmi imagename`
- 强制删除本地镜像
`docker rmi -f imagename`

### 容器
- 创建容器
`docker run -it --name mycentos centos /bin/bash`  
`-it` 交互式运行  
`-d` 后台运行  
`--rm` 容器停止后删除
- 启动容器
`docker start mycentos`
- 停止容器
`docker stop mycentos`
- 重启容器
`docker restart mycentos`
- 删除容器
`docker rm mycentos`
- 强制删除容器
`docker rm -f mycentos`
- 查看容器
`docker ps -a`
- 查看最近创建容器
`docker ps -l`
- 进入容器
    - 根据容器id或名称  
    `docker attach mycentos`  
    `docker exec -it mynginx /bin/bash`  
    - 根据pid进入容器  
    获得pid  
    `docker inspect --format "{{.State.Pid}}" mynginx`  
    进入容器，pid替换成上一步获得的pid  
    `nsenter --target pid --mount --uts --ipc --net --pid`
- 退出容器  
`exit`会导致容器停止（根据pid进入则不会）  
`ctrl+p+q`容器保持运行
- 查看容器/镜像的元数据  
`docker inspect mynginx` 查看容器元数据  
`docker inspect centos` 查看镜像元数据

## 网络
### 网络名称空间
1. 检查是否有`iproute`包  
`rpm -q iproute`
2. 添加网络名称空间  
`ip netns add r1`  
`ip netns add r2`
3. 查看网络名称空间  
`ip netns list`
4. 网络名称空间中执行命令  
`ip netns exec r1 ifconfig`
5. 创建虚拟网卡对，分配给网络名称空间  
    1. 创建虚拟网卡对  
    `ip link add name veth1.1 type veth peer name veth1.2`
    2. 分配给网络名称空间  
    `ip link set dev veth1.2 netns r1`
    `ip link set dev veth1.1 netns r2`
    3. 查看名称空间中的网卡  
    `ip netns exec r1 ifconfig -a`
    `ip netns exec r2 ifconfig -a`
    4. 修改名称空间中的网卡名  
    `ip netns exec r1 ip link set dev veth1.2 name eth0`
6. 查看网卡  
`ip link show`
7. 激活网卡  
`ip netns exec r1 ifconfig eth0 10.1.0.2/24 up` 激活名称空间r1中的网卡  
`ip netns exec r2 ifconfig veth1.1 10.1.0.1/24 up` 激活名称空间r2中的网卡
8. 测试名称空间互相通信  
`ip netns exec r2 ping 10.1.0.2`

### docker 容器网络
1. 封闭式容器  
`docker run --name t1 --network none --rm busybox:latest`
2. 桥接式容器  
`docker run --name t1 --network bridge -h t1.mervyn.com --dns 114.114.114.114 --dns-search ilinux.io --add-host www.baidu.com:1.1.1.1 --rm busybox:latest`  
`-h`设定主机名Hostname  
`--dns`指定dns  
`--dns-search`设定容器的搜索域  
`--add-host`注入hosts文件  
3. 联盟式容器  
`docker run --name b1 -it --rm busybox`  
`docker run --name b2 --network container:b1 -it --rm busybox` 共享b1的网络名称空间  
`docker run --name b3 --network host -it --rm busybox` 共享宿主机的网络名称空间

## 端口映射
- 随机端口  
`docker run -P -d --name mynginx nginx`  
随机映射所有端口
- 指定端口  
    - 映射指定端口到任意端口  
    `docker run -d -p containerPort --name mynginx nginx`  
    - 映射到指定端口  
    `docker run -d -p hostPort:containerPort --name mynginx nginx`  
    - 映射到指定地址的指定端口  
    `docker run -d -p ip:hostPort:containerPort --name mynginx nginx` 
    - 映射到指定地址的任意端口  
    `docker run -d -p ip::containerPort --name mynginx nginx`
    - 指定多端口  
    `docker run -d -p hostPort:containerPort -p hostPort:containerPort --name mynginx nginx`
- 查看端口映射情况
`docker port mynginx`

可使用`宿主机ip:映射端口`访问  

## 数据卷
### 数据卷
#### 特性
- docker数据卷是独立于容器的存在，与容器的生存周期分离。
- 存在于宿主机文件系统中。
- 可以是目录也可以是文件。
- docker容器可以利用数据卷的技术与宿主机进行数据共享。
- 同一个目录或者文件可以支持多个docker容器的访问。
- 对数据卷的修改会立即生效。
- 修改数据卷不影响镜像。
- 
#### 命令
- 挂载宿主机目录作为数据卷  
`docker run -it -v /datavolume:/data centos /bin/bash` `/datavolume`为宿主机目录，`/data`为容器目录  
进入容器，在容器目录下创建文件并修改  
```
touch /data/file
echo "i'm in container" > /data/file
```
退出容器，查看宿主机目录，发现文件内容已同步修改。
- 挂载宿主机文件作为数据卷  
`docker run -it -v /mervyn/datavolume/file:/data/file centos /bin/bash`  
进入容器，在目录下修改file文件  
`echo "i'm in container2" > /data/file`  
创建文件file2并修改
```
touch /data/file2
echo "file2 in container" > /data/file2
```
退出容器，查看宿主机目录，发现file文件内容已同步修改，而file2文件并未创建。
- 挂载数据卷权限为只读  
`docker run -it -v /datavolume:/data:ro centos /bin/bash`

### 数据卷容器
容器挂载数据卷，其他容器挂载该容器实现数据共享，挂载数据卷的容器，就叫做数据卷容器。

1. 创建容器挂载数据卷，作为数据卷容器  
`docker run -it --name volume_container -v /datavolume:/data centos /bin/bash`
2. 创建容器挂载数据卷容器  
`docker run -it --name volume_centos_1 --volumes-from volume_container centos /bin/bash`  
`docker run -it --name volume_centos_2 --volumes-from volume_container centos /bin/bash`  
在volume_centos_1、volume_centos_2中的`/data`目录增删改文件，均会同步，实现数据共享。

即使删除数据卷容器，已挂载数据卷容器的容器仍然可以访问数据卷容器挂载的数据卷。

### 数据卷的备份和还原
- 备份  
创建一个用于备份数据的容器，同时挂载需要备份的数据卷（数据卷容器）和备份存放的目录，并执行压缩或者复制命令  
`docker run --volumes-from volume_container -v /backup:/backup --name volume_backup centos tar cvf /backup/backup.tar /datavolume`
- 还原  
创建一个用于还原数据的容器，同时挂载需要还原数据的数据卷（数据卷容器）和备份存放的目录，并执行解压或者复制命令  
`docker run --volumes-from volume_container -v /backup:/backup --name volume_restore centos tar xvf /backup/backup.tar -C /`

## 创建镜像
### 手动创建
1. 使用已有镜像创建容器  
`docker run -it --name centos_manural centos /bin/bash`
2. 修改容器
3. 退出容器
4. 提交修改的副本为镜像  
`docker commit -m "my centos manural" centos_menural mervyn/my-centos:v1`
5. 使用新镜像创建容器
`docker run -it mervyn/my-centos:v1`

### Dockerfile
Dockerfile 由一行行命令语句组成，并且支持以 # 开头的注释行。  
一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。
#### Dockerfile指令
- `FROM` 指定基础镜像，必须是第一条命令  
- `MAINTAINER` 镜像作者信息  
- `RUN` 当前镜像基础上执行指定命令，并提交为新的镜像  
- `CMD` 指定启动容器时执行的命令  
每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。  
如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令
- `EXPOSE` 暴露的端口号，供互联系统使用
- `ENV` 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持
- `ADD` 复制本地文件到容器中，对于tar文件会自动解压。
- `COPY` 仅复制本地文件到容器中
- `ENTRYPOINT` 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖
- `VOLUME` 挂载数据卷
- `USER` 指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户
- `WORKDIR` 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录
- `ONBUIL` 配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令

编写完Dockerfile后执行`docker build -t mervyn/centos-by-df:v1 .`命令构建镜像。  
`-t`为指定标签，最后的`.`为指定Dockerfile在当前工作目录下，也可使用绝对路径。

## Registry 私有镜像仓库
1. 下载registry  
`docker pull registry`
2. 启动registry容器  
`docker run -d -p 5000:5000 --name registry registry`
3. 为镜像打标签  
`docker tag mervyn/my-centos:v1 192.168.177.129:5000/my-centos:v1`
4. 上传到镜像服务器  
**docker daemon要求docker registry必须是https的，如果不是，需要告诉docker deamon该registry可用。**  
修改文件`/etc/docker/daemon.json`若无此文件则创建  
```
{
  "registry-mirrors": [ "https://registry.docker-cn.com"],
  "insecure-registries": [ "192.168.177.129:5000"]    #增加此行
}
```
刷新，重启docker服务  
```
systemctl daemon-reload
systemctl restart docker
```
`docker push 192.168.177.129:5000/my-centos:v1`  
5. 在其他服务器pull镜像  
修改文件`/etc/docker/daemon.json`同上。  
`docker pull 192.168.177.129:5000/my-centos:v1`

## 企业级仓库Harbor
以下内容以`Harbor_v1.9.4`版本为例

1. 下载  
```
wget https://github.com/goharbor/harbor/releases/download/v1.9.4/harbor-offline-installer-v1.9.4.tgz
tar -xvf harbor-offline-installer-v1.9.4.tgz -C /usr/local/
```
2. 编辑配置文件  
```
cd /usr/local/harbor
vim harbor.yml
```
配置文件详见[Harbor Installation and Configuration Guide](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)
3. 安装  
    1. 安装`docker-compose`  
    ```
    curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ```
    2. 执行安装脚本`./install.sh`
4. 访问webUI  
    1. 访问地址为`harbor.yml`中配置的`hostname`  
    比如我的为：`http://192.168.177.129`
    2. 登陆admin  
    密码为`harbor.yml`中配置的`harbor_admin_password`
    3. 新建项目、用户等自行操作
5. 修改daemon.json  
`vim /etc/docker/daemon.json`  
```
{
  "registry-mirrors": [ "https://registry.docker-cn.com"],
  "insecure-registries": ["192.168.177.129:5000","192.168.177.129"]    #registry5000端口， harbor80端口
}
```
```
systemctl daemon-reload
systemctl restart docker
```
6. 登陆harbor  
`docker login 192.168.177.129`
7. push镜像
    1. 打tag  
    ```
    docker tag mervyn/my-centos:v1 192.168.177.129/test/my-centos:v1
    docker tag mervyn/my-centos:v2 192.168.177.129/test/my-centos:v2
    ```
    2. push  
    `docker push 192.168.177.129/test/my-centos`  
    tag可省略，将所有版本push。
8. 暂停、重启harbor(会找到当前目录`docker-compose.yml`文件)  
`docker-compose pause`  
`docker-compose unpause` 
9. 停止、启动harbor(会找到当前目录`docker-compose.yml`文件)  
`docker-compose stop`  
`docker-compose start`  

## 资源限制
### 内存
- `-m`或`--memory=` 限制可使用ram内存  
最低允许值为`4m`  
使用超过设置的值，进程可能会被杀掉
- `--memory-swap`  
**必须先设定`-m`**  
假设`--memory`设值M
    - `--memory-swap`设值S  
    容器可用总空间为S，其中ram为M，swap为(S-M)，若S=M，则无可用swap资源
    - `--memory-swap`设值0，或不设值  
    若主机启用了swap，则容器可用swap为2*M
    - `--memory-swap`设值-1  
    若主机启用了swap，则容器可使用最大为主机上所有swap空间资源
- `--memory-swappiness` 容器可使用交换资源的倾向性  
设值0，能不用则不用（并不是禁用）  
设值100，能用时就用
- `--oom-kill-disable` 不希望因为内存耗尽而被杀掉

### CPU
- `--cpu-shares` 实现各进程按比例分配cpu  
假设A进程1024，B进程512，C进程2048  
当B,C空闲时，A可占满所有cpu  
当C空闲时，A、B按2:1的比例分配cpu  
当A,B,C都忙碌时，按2:1:4分配cpu  
- `--cpus` 最多可以使用多少cpu  
假设机器为4核cpu，参数设为2  
则可以使用0,1两个达到2，或者1,3两核达到2等等  
也可以使用0,1,2,3各0.5达到2  
- `--cpuset-cpus` 指定允许使用哪个cpu  
设值0-3则表示允许使用4个cpu  
设值1,2 则表示仅允许使用1,2两个cpu
