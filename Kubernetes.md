[TOC]
# Kubernetes
## 组件
- master
    - controller-manager
    - scheduler
    - apiserver
- node
    - kubelet
    - kube-proxy
    - docker
## 部署
### 部署方式
- 传统方式部署k8s自身，k8s自己的相关组件统统运行为系统级的守护进程，包括master和node上的组件。缺点：每一步都需要手动解决，包括做证书等……过程繁琐且复杂；系统级的守护进程某个组件挂了需要手动启动。
- kubeadm
    - 把k8s自己部署为pod
    - master和node只需有手动安装kubelet和docker
    - master和node中的组件运行为pod（static pod）
    - flannel运行为pod，托管在k8s集群中 
### kubeadm部署
#### 前提
- 各节点时间同步
- 各节点主机名称解析
    ```
    hostname master
    hostnamectl set-hostname master
    ```
    ```
    vim /etc/hosts
    192.168.xx.xx master
    192.168.xx.xx node01
    192.168.xx.xx node02
    #等等
    ```
- 各节点iptables及firewalld服务器被disabled
    ```
    systemctl stop firewalld
    systemctl disable firewalld
    ```
- 关闭selinux
    ```
    sed -i 's/enforcing/disabled/' /etc/selinux/config
    setenforce 0
    ```
- 关闭swap
    ```
    swapoff -a  # 临时
    vim /etc/fstab  # 永久
    #注释掉swap行
    ```

#### 步骤
1. 设置各节点安装程序包
    1. 下载docker仓库文件  
        `cd /etc/yum.repos.d/`  
        `wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
    2. 生存Kubernetes仓库文件  
        `vim kubernetes.repo`  
        ```
        [kubernetes]
        name=Kubernetes Repo
        baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        ```
    3. 安装  
        `yum install docker-ce kubelet kubeadm kubectl -y` node节点可不安装kubectl
2. 初始化主节点
    1. 设置代理，获取kubernetes所需镜像（我试过无效，最终采用添加`--image-repository`的方式从阿里云获取镜像）  
        `vim /usr/lib/systemd/system/docker.service`  
        `[Service]`下`ExecStart`前添加  
        ```
        Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
        Environment="NO_PROXY=127.0.0.0/8,172.20.0.0/16"
        ```
        重启docker并查看
        ```
        systemctl daemon-reload
        systemctl start docker
        docker info
        ```
    2. 设置ip6tables和iptables值为1
        ```
        echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
        echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
        ```
        如果提示`No such file or directory`
        ```
        #安装bridge-util软件，加载bridge模块，加载br_netfilter模块
        yum install -y bridge-utils
        modprobe bridge
        modprobe br_netfilter
        ```
    3. 设置kubelet、docker开机自启动  
        ```
        systemctl enable kubelet docker
        ```
    4. 编辑kubelet配置文件`/etc/sysconfig/kubelet`，设置忽略Swap启动的状态错误  
        `KUBELET_EXTRA_ARGS="--fail-swap-on=false"`
    5. 启动docker、kubelet
        ```
        systemctl start docker kubelet
        ```
    6. 使用`kubeadm init`初始化  
        `kubeadm init --kubernetes-version=v1.17.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap --image-repository registry.aliyuncs.com/google_containers`  
        记录生成的`kubeadm join`完整命令  
        若失败，需重新初始化，则重置kubeadm  
        `kubeadm reset`
    7. 初始化kubectl  
        ```
        mkdir -p $HOME/.kube
        cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        ```
    8. 部署flannel  
        `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`  
        如果下载失败：
            1. 从其他服务器下载镜像，修改tag，再打包
            ```
            docker pull lizhenliang/flannel:v0.11.0-amd64
            docker images
            docker tag lizhenliang/flannel:v0.11.0-amd64 quay.io/coreos/flannel:v0.11.0-amd64
            docker image save quay.io/coreos/flannel:v0.11.0-amd64 -o flannelv0.11.0-amd64.tar
            ```
            2. 导入镜像到服务器上，（master和nodes服务器都要）  
            `docker load -i flannelv0.11.0-amd64.tar`
            3. 再执行该命令  
            `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`  
            如果之前安装过，没有成功，则先删除  
            `kubectl delete -f kube-flannel.yml`  
            查看flannel是否成功启动  
            `kubectl get pods -n kube-system`
    9. 验证master节点  
        `kubectl get nodes`
3. 添加node节点到集群中
    1. 同上操作：
        1. 启动docker
        2. 设置docker、kubelet开机自启
        3. 设置ip6tables、iptables值为1
        4. 配置kubelet忽略Swap启动状态错误
        5. 导入镜像
    2. 将节点加入master集群中  
        使用前面记录的`kubeadm join`命令， 并在后面加上`--ignore-preflight-errors=Swap`
    3. 回到master节点验证  
        `kubectl get nodes`

## kubectl命令
- 查看节点详细信息  
    `kubectl describe node k8s-node1`
- 创建pod  
    `kubectl run nginx-deploy --image=nginx --port=80 --replicas=1`  
    `nginx-deploy`名称  
    `--image=nginx`指定镜像  
    `--port=80`暴露端口  
    `--replicas=1`创建副本数  
- 查看pod信息  
    `kubectl describe pods nginx-deploy-98c9d6c66-tvsqc` 详细信息  
    `kubectl get pods -o wide` 获取pods运行在哪个节点上  
    `kubectl get pods -n kube-system` 查看`kube-system`名称空间的pods  
    `kubectl logs nginx-deploy-98c9d6c66-tvsqc` 查看pod运行日志
- 删除pod  
    `kubectl delete pods nginx-deploy-98c9d6c66-tvsqc`  
    再执行`kubectl get pods`会发现新的`nginx-deploy`生成了，pod管理确保pod副本数量严格符合用户定义，即上述`--replicas`
- 为pod资源创建一个service  
    `kubectl expose deployment nginx-deploy --name=nginx --port=80 --target-port=80 --protocol=TCP`  
    为pod提供固定访问端点
- 查看service  
    `kubectl get svc`
- 创建pod客户端  
    `kubectl run client --image=busybox --replicas=1 -it --restart=Never`  
    客户端内可直接使用第5步定义的名称访问pod：通过label-selector
- 动态扩缩容  
    `kubectl scale --replicas=5 deployment nginx-deploy`
- 更新镜像  
    `kubectl set image deployment nginx-deploy nginx-deploy=nginx:1.15`
- 回滚  
    `kubectl rollout undo deployment nginx-deploy`
- 修改service属性  
    `kubectl edit svc nginx`  
    将`type`修改为`NodePort`。  
    查看service  
    `kubectl get svc`  
    可以看到nginx的ports多了个80外的端口  
    即可在集群外部访问`节点ip:新的port`。  
- 查看pod的yaml信息  
    `kubectl get pod nginx-deploy-68875965fc-dw22b -o yaml`
- 查看创建pod的yaml字段  
    `kubectl explain pods`  
    内嵌字段使用`kubectl explain pods.metadata`
- 通过yaml文件创建资源  
    `kubectl create -f pod-demo.yaml `
- 给资源打标签  
    `kubectl label pods pod-demo release=canary`  
    如果是修改标签  
    `kubectl label pods pod-demo release=stable --overwrite`
- 选择标签
    - `kubectl get pods -l`  
        - 等值关系：
            - `=`, `kubectl get pods -l release=stable` 选择所有有release=stable标签的资源
            - `==`, 同上
                + `!=`, `kubectl get pods -l release!=stable` 选择所有release!=stable标签的资源，包括没有release标签的资源
        - 集合关系：
            - `key in (value1, value2, ...)` 选择所有key=value1或key=value2或...的资源
            - `key notin(value1, value2, ...)` 选择所有key!=value1且key!=value2且...的资源  
            - `key` 选择所有有key标签的资源
            - `!key` 选择所有没有key标签的资源
    - `kubectl get pods -L key` 显示所有key标签的值

## yaml配置清单
- `apiVersion` api版本  
    `group/version`  
    `kubectl api-versions`可获取
- `kind` 资源类别
- `metadata` 元数据
    - `name` 统一类别中name唯一
    - `namespace` 名称空间
    - `labels` 标签  
        `key` 字母、数字、_、-、.  
        `value` 可以为空，只能字母或数字开头及结尾，中间可使用字母、数字、_、-、.
- `spec` 期望目标状态， 不同资源类型的spec字段内容不尽相同
    - `container` 容器
        - `name` 容器名称
        - `image` 容器镜像
        - `imagePullPolicy` 拉取镜像策略  
            `Always` 总是自动拉取  
            `Never` 从不自动拉取（需要用户手动拉取）  
            `IfNotPresent` 如果存在该镜像则不拉取
        - `port` 暴露端口
            - `containerPort` 需要暴露的容器端口
        - `command`、`args` 修改镜像中的默认设置  
            `command` 容器的启动命令列表，相较于DockerFile的`Entrypoint`字段  
            `args` 容器的启动命令参数列表，相较于DockerFile的`Cmd`字段
            - 如果`command`、`args`均不指定，则默认使用Docker镜像定义的`Entrypoint`和`Cmd`
            - 指定了`command`没指定`args`，只运行`command`，忽略`Entrypoint`和`Cmd`
            - 指定了`args`没指定`command`，只运行`Entrypoint`并使用`args`参数
            - 如果`command`、`args`均指定，使用`args`参数运行`command`
    - `nodeSelector` 节点标签选择器  
        `key: value` 运行在拥有`key=value`标签的节点上
    - `nodeName` 指定节点
    - `selector` 适用于`kind=deployment`的资源，把所有正在运行的，拥有给定标签的pod识别为被管理的对象。
        - `matchLabels` 直接给定键值
        - `matchExpressions` 基于表达式定义选择器。  
            `{key:"KEY", operator:"OPERATOR", values:[val1, val2, ...]}`  
            当`operator`为：
            - `In`, `NotIn`时，`values`字段非空
            - `Exists`, `NotExists`时，`values`字段为空
    - `restartPolicy` Pod内所有容器的重启策略
        - `Always` 总是重启
        - `OnFailure` 出错时重启
        - `Never` 从不重启
- `status` 当前状态，无限向期望状态靠近，由kubernetes集群维护，用户不能定义

## Pod生命周期
[Kubernetes Pod 生命周期 - 简书](https://www.jianshu.com/p/91625e7a8259?utm_source=oschina-app)
### 状态
- `Pending`, Pod 已被 Kubernetes 接受，但尚未创建一个或多个容器镜像。这包括被调度之前的时间以及通过网络下载镜像所花费的时间，执行需要一段时间。
- `Running`, Pod 已经被绑定到了一个节点，所有容器已被创建。至少一个容器正在运行，或者正在启动或重新启动。
- `Failed`, 所有容器终止，至少有一个容器以失败方式终止。也就是说，这个容器要么已非 0 状态退出，要么被系统终止。
- `Succeeded`, 所有容器成功终止，也不会重启。
- `Unkown`, 由于一些原因，Pod 的状态无法获取，通常是与 Pod 通信时出错导致的

### 容器探测
- `livenessProbe`：指示容器是否正在运行，如果活动探测失败，则 kubelet 会杀死容器，并且容器将受其重启策略的约束。如果不指定活动探测，默认状态是 Success。
- `readinessProbe`：指示容器是否已准备好为请求提供服务，如果准备情况探测失败，则控制器会从与 Pod 匹配的所有服务的端点中删除 Pod 的 IP 地址。初始化延迟之前的默认准备状态是 Failure，如果容器未提供准备情况探测，则默认状态为 Success。
#### 探针类型
- ExecAction：在容器内部执行指定的命令，如果命令以状态代码 0 退出，则认为诊断成功。
- TCPSocketAction：对指定 IP 和端口的容器执行 TCP 检查，如果端口打开，则认为诊断成功。
- HTTPGetAction：对指定 IP + port + path路径上的容器的执行 HTTP Get 请求。如果响应的状态代码大于或等于 200 且小于 400，则认为诊断成功。