[TOC]

<!--开始学习时间2020-->

# 特性

- 自动装箱：基于容器对应用环境的资源配置要求自动部署应用容器
- 自动修复：
    - 容器失败时自动重启
    - 节点出问题时，重新调度、部署
    - 容器未通过检查时，关闭容器
    - 容器正常运行，才对外提供服务
- 水平扩展：对应用容器进行规模扩大、裁剪
- 服务发现：包含服务发现和负载均衡
- 滚动更新：根据应用的变化、对应用进行一次性或批量式更新
- 版本回退：根据应用部署情况，对应用进行历史版本即时回退
- 密钥和配置管理：不需要重新构建镜像，更新密钥和应用配置。
- 存储编排：
    - 自动实现存储系统挂载及应用
    - 存储卷动态供给

# 组件

- master
    - apiserver：负责接收并处理请求。各组件协调者，以RESTful API提供接口服务，所有对象资源的增删改查和监听操作都交给APIServer处理后再提交给Etcd存储
    - scheduler：调度容器创建的请求
    - controller-manager：检测控制器健康、确保容器健康
    - etcd：分布式键值存储，用于保存集群状态数据，比如pod、service等
- node
    - kubelet：用于与master通信，接收master调度过来的任务并执行。管理本机容器的生命周期。将每个pod转换成一组容器。
    - kube-proxy：在Node节点上实现Pod网络代理。
    - docker：容器引擎，用于创建、运行容器

# 部署

## 部署方式

- 传统方式部署k8s自身，k8s自己的相关组件统统运行为系统级的守护进程，包括master和node上的组件。缺点：每一步都需要手动解决，包括做证书等……过程繁琐且复杂；系统级的守护进程某个组件挂了需要手动启动。
- kubeadm
    - 把k8s自己部署为pod
    - master和node只需有手动安装kubelet和docker
    - master和node中的组件运行为pod（static pod）
    - flannel运行为pod，托管在k8s集群中 

## 使用 kubeadm 部署 Kubernetes

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

```bash
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

### 学习目标

1. 在所有节点上安装Docker和kubeadm
2. 部署Kubernetes Master
3. 部署容器网络插件
4. 部署 Kubernetes Node，将节点加入Kubernetes集群中
5. 部署Dashboard Web页面，可视化查看Kubernetes资源


### 前提

- 各节点时间同步
- 各节点主机名称解析
    ```bash
    hostname master
    hostnamectl set-hostname master
    echo "127.0.0.1   $(hostname)" >> /etc/hosts
    ```
- 各节点iptables及firewalld服务器被disabled
    ```bash
    systemctl stop firewalld
    systemctl disable firewalld
    ```
- 关闭selinux
    ```bash
    sed -i 's/enforcing/disabled/' /etc/selinux/config
    setenforce 0
    ```
- 关闭swap
    ```bash
    swapoff -a  # 临时
    vim /etc/fstab  # 永久
    #注释掉swap行
    ```

### 步骤

1. 设置各节点安装程序包
    1. 下载docker仓库文件
        ```bash
        cd /etc/yum.repos.d/
        wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
        ```
        
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
        
    3. 安装  （指定版本）
       
        ```bash
        yum install -y docker-ce-19.03.8 kubelet-1.18.8 kubeadm-1.18.8 kubectl-1.18.8
        ```
        
        node节点可不安装kubectl
2. 初始化主节点
    1. 设置ip6tables和iptables值为1
        ```bash
        echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
        echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
        ```
        如果提示`No such file or directory`
        ```bash
        #安装bridge-util软件，加载bridge模块，加载br_netfilter模块
        yum install -y bridge-utils
        modprobe bridge
        modprobe br_netfilter
        ```
    2. 设置kubelet、docker开机自启动
        ```bash
        systemctl enable kubelet docker
        ```
    3. 启动docker、kubelet
        ```bash
        systemctl start docker kubelet
        ```
    4. 使用`kubeadm init`初始化
        ```bash
        kubeadm init \
        --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
        --kubernetes-version=v1.18.8 \
        --service-cidr=10.1.0.0/16 \
        --pod-network-cidr=10.244.0.0/16
        ```
        记录生成的`kubeadm join`完整命令
        若失败，需重新初始化，则重置kubeadm
        `kubeadm reset`
    5. 初始化kubectl
        ```bash
        mkdir -p $HOME/.kube
        cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        ```
    6. 部署flannel
        ```bash
        kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
        ```
        如果下载失败：
        1. 从其他服务器下载镜像，修改tag，再打包
            ```bash
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
    7. 验证master节点
        `kubectl get nodes`
3. 添加node节点到集群中
    1. 同上操作：
        1. 启动docker
        2. 设置docker、kubelet开机自启
        3. 设置ip6tables、iptables值为1
        4. 导入镜像
    2. 将节点加入master集群中
        使用前面记录的`kubeadm join`命令
    3. 回到master节点验证
        `kubectl get nodes`

# Kubernetes 中的资源

- 工作负载型资源 :
    - Pod
    - ReplicaSet
    - Deployment
    - StatefulSet
    - DaemonSet
    - Job
    - Cronjob
    - ...
- 服务发现及均衡
    - Service
    - Ingress
    - ...
- 配置与存储
    - Volume
    - ConfigMap
    - Secret
    - ...
- 集群级资源
    - Namespace
    - Node
    - Role
    - RoleBinding
    - ...
- 元数据型资源
    - HPA
    - PodTemplate
    - LimitRange
    - ...

# kubectl命令

kubectl是api server的客户端程序
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
- 通过yaml文件删除资源
    `kubectl delete -f pod-demo.yaml`
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

- 显示所有key标签的值

    `kubectl get pods -L key`

- 通过文件对资源应用配置

    `kubectl apply -f demo.yaml`

    首次执行为创建

    多次执行为更新

- 更新资源字段、打补丁

    `kubectl patch deployment deploy-demo -p '{"spec":{"replicas":5}}' `

![20200821103335](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071633194.jpg)

# yaml配置清单

**创建资源的方法**

apiserver 仅接收JSON格式的资源定义。
yaml 格式提供配置清单，apiserver可自动将其转换为json格式，然后再执行。

**配置清单字段**

使用`kubectl explain TYPE`命令获取各类型资源的字段
- `apiVersion` api版本
    `group/version`
    `kubectl api-versions`可获取
    
- `kind` 资源类别

- `metadata` 元数据
    - `name` 同一类别中name唯一

    - `namespace` 名称空间

    - `labels` 标签
        `key` 字母、数字、_、-、.
        `value` 可以为空，只能字母或数字开头及结尾，中间可使用字母、数字、_、-、.
        
    - `annotations`注解

        与`label`不同的地方在于，它不能用于挑选资源对象，仅用于为对象提供“元数据”。
    
- `spec` 期望目标状态， **不同资源类型的spec字段内容不尽相同**
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
            
            ![image-20200825091158807](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071633602.png)
            
        - `livenessProbe`存活性探测

            - `exec`、`httpGet`、`tcpSocket`三种探针类型
            - `failureThreshold` 被认为失败的最少连续失败次数
            - `initialDelaySeconds` 容器启动后、探针初始化前的时间
            - `periodSeconds` 探针频率
            - `successThreshold` 被认为成功的最少连续成功次数
            - `timeoutSeconds`超时时间

        - `readinessProbe`就绪性探测

        - `lifecycle` 生命周期回调

            - `postStart` 容器创建后立刻执行，如果失败，容器将停止并根据重启策略重启容器。
            - `preStop` 结束前回调，执行于以下情况引起的容器终止
                - API 请求
                - 管理事件
                  - 探测失败
                  - 抢占
                  - 资源争夺
                  - ...
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

# Pod

## Pod生命周期

![20200827092315](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071634568.jpg)

生命周期中的重要行为：

1. 初始化容器
2. 生命周期回调
3. 容器探测

### 状态

- `Pending`, Pod 已被 Kubernetes 接受，但尚未创建一个或多个容器镜像。这包括被调度之前的时间以及通过网络下载镜像所花费的时间，执行需要一段时间。
- `Running`, Pod 已经被绑定到了一个节点，所有容器已被创建。至少一个容器正在运行，或者正在启动或重新启动。
- `Failed`, 所有容器终止，至少有一个容器以失败方式终止。也就是说，这个容器要么已非 0 状态退出，要么被系统终止。
- `Succeeded`, 所有容器成功终止，也不会重启。
- `Unkown`, 由于一些原因，Pod 的状态无法获取，通常是与 Pod 通信时出错导致的

### 容器探测

- `livenessProbe`：指示容器是否正在运行，如果活动探测失败，则 kubelet 会杀死容器，并且容器将受其重启策略的约束。如果不指定活动探测，默认状态是 Success。

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: liveness-exec-pod
    namespace: default
  spec:
    containers:
    - name: liveness-exec-container
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: ["/bin/sh", "-c", "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 3600"]
      livenessProbe:
        exec:
          command: ["test", "-e", "/tmp/healthy"]
        initialDelaySeconds: 1
        periodSeconds: 3
  ```

- `readinessProbe`：指示容器是否已准备好为请求提供服务，如果准备情况探测失败，则控制器会从与 Pod 匹配的所有服务的端点中删除 Pod 的 IP 地址。初始化延迟之前的默认准备状态是 Failure，如果容器未提供准备情况探测，则默认状态为 Success。

#### 探针类型

- `ExecAction`：在容器内部执行指定的命令，如果命令以状态代码 0 退出，则认为诊断成功。
- `TCPSocketAction`：对指定 IP 和端口的容器执行 TCP 检查，如果端口打开，则认为诊断成功。
- `HTTPGetAction`：对指定 IP + port + path路径上的容器的执行 HTTP Get 请求。如果响应的状态代码大于或等于 200 且小于 400，则认为诊断成功。

### 生命周期回调

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:            #在postStart启动时，ENTRYPOINT有可能还没结束
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:              #发生在容器被杀死之前，它会阻塞当前的容器杀死进程，直到这个Hook定义操作完成之后，才允许容器被杀死
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]
```

## Pod 控制器

### ReplicaSet

`ReplicaSet`确保`Pod`资源一直符合用户期望的`replicas`数量的状态。

**核心资源**

1. 用户期望副本数，确保所控制的资源满足用户期望。
2. 标签选择器，控制指定的资源。
3. Pod 资源模板，当资源不满足时，根据资源模板完成创建。

> A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.
>
> ReplicaSet可确保指定数量的pod“replicas”在任何设定的时间运行。然而，Deployments是一个更高层次的概念，它管理ReplicaSets，并提供对pod的声明性更新以及许多其他的功能。因此，我们建议您使用Deployments而不是直接使用ReplicaSets，除非您需要自定义更新编排或根本不需要更新。

**配置清单**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: rs-demo
    namespace: default
spec:
    replicas: 2 # 副本数
    selector: # 托管pod的label选择器
        matchLabels:
            app: rs-demo-pod
            release: stable
    template: # 模板，控制器会根据模板创建pods
        metadata:
            name: rs-demo-pod # 无用字段，pod会以replicaSet的名称+'_随机码'命名
            labels: # 需与replicaSet的selector相符
                app: rs-demo-pod
                release: stable
                enviroment: qa
        spec:
            containers:
            - name: rs-demo-container
              image: nginx
              imagePullPolicy: IfNotPresent
              ports:
              - name: http
                containerPort: 80
```

**扩容缩容**

```bash
# 使用edit命令
kubectl edit rs rs-demo
# 修改replicas的值即可

# 使用scale命令
kubectl scale --replicas=2 rs rs-demo
```

**更新镜像版本**

```bash
kubectl edit rs rs-demo
# 修改了模板，但已有pod镜像并不会更新，因为副本数目正确，pod不会重建。后续重建的pod才会使用更新的镜像。
```

![20200901161504](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071634191.jpg)

### Deployment

`Deployment`建构在`ReplicaSet`上，不直接控制`Pod`，而是通过控制`ReplicaSet`来控制`Pod`。只用于管控无状态应用。

**作用**

1. 创建部署`Pod`和`ReplicaSet`
2. 扩容、缩容
3. 滚动更新、回滚应用
4. 暂停和继续`Deployment`

**`Deployments` - `ReplicaSets` - `Pods` 结构**

![20200901164019](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071634511.jpg)

创建`Deployment`后，查询`ReplicaSet`发现自动创建了`ReplicaSet`

```bash
[root@master ~]# kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
deploy-demo   2/2     2            2           4m12s
[root@master ~]# kubectl get rs
NAME                     DESIRED   CURRENT   READY   AGE
deploy-demo-7b5b46db88   2         2         2       4m16s
# rs的后缀随机码为模板的hash值，以便追踪到关联到的模板创建pods
[root@master ~]# kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
deploy-demo-7b5b46db88-hnflh   1/1     Running   0          4m20s
deploy-demo-7b5b46db88-mc6z4   1/1     Running   0          4m20s
```

**配置清单**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-deploy
    namespace: default
spec:
    replicas: 3
    selector:
        matchLabels:
            app: myapp
            release: stable
    strategy: # 更新策略
        type: RollingUpdate # 更新方式
        rollingUpdate: # 滚动更新策略
            maxSurge: 1 # 在滚动中，可以创建多少个新pod
            maxUnavailable: 0 # 在滚动中，可以删除多少个旧pod
    template:
        metadata:
            name: myapp-deploy-pod
            namespace: default
            labels:
                app: myapp
                release: stable
        spec:
            containers:
            - name: myapp-deploy-container
              image: ikubernetes/myapp:v1
              imagePullPolicy: IfNotPresent
              ports:
              - name: http
                containerPort: 80

```

**滚动更新**

```bash
kubectl edit deployment deploy-demo
kubectl set image deployment deploy-demo deploy-demo-container=nginx
```

```bash
# 查看更新历史
kubectl rollout history deployment deploy-demo
# 回滚至指定版本(不指定则默认上一版本)
kubectl rollout undo deployment deploy-demo --to-revision=1
```

### DaemonSet

用于确保集群中每一个节点运行且只运行一个特定的Pod副本。常用来部署一些集群的日志、监控或者其他系统管理应用。

当节点新加入集群时，会自动添加这些Pod到这些节点。

当节点从集群中删除时，这些Pod会被删除。

常用于：

1. 日志收集
2. 系统监控
3. 系统程序

### Job

`Job`负责处理一次性任务。

`Pod`任务完成后退出，不需要作为守护进程运行。

重启策略为：仅当任务未完成的异常退出时重启。

### CronJob

`CronJob`周期性运行任务，即定时任务，重复创建`Job`来执行任务。

### StatefulSet

管理有状态应用（对应Deployments和ReplicaSets是为无状态应用而设计），并且每个副本被单独管理。

常用于持久化存储。

# Service

`Service`为`Pod`提供固定访问端点。避免`Pod`重启后`ip`改变无法访问。

`Service`通过`selector`关联到对应的`Pods`

**代理模式**

- `userspace`

  效率低

  ![20200909165554](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071634231.jpg)

- `iptables`

  灵活、功能强大

  规则遍历匹配和更新

  可扩展性

  ![20200909165818](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071634214.jpg)

- `IPVS`

  工作在内核态，有更好的性能

  调度算法丰富
  
  ![20200909165241](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071635866.jpg)

**DNS资源记录**

`DNS`服务监视`Kubernetes API`，为每个`Service`创建`DNS`记录用于域名解析。

可通过`DNS`名称访问`Pods`

`SVC_NAME.NAMESPACE_NAME.svc.cluster.local`

**类型**

- `ClusterIP`集群内部通讯
- `NodePort`对外暴露
- `LoadBalancer`对外暴露和负载均衡，仅限部分云平台

**配置清单**

```yaml
apiVersion: v1
kind: Service
metadata:
    name: myapp 
    namespace: default
spec:
    selector:
        app: myapp
        release: stable
    type: NodePort
    ports: 
    - port: 80
      targetPort: 80
```

# 参考资料

[kubeadm故障排查](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

[Kubernetes Pod 生命周期 - 简书](https://www.jianshu.com/p/91625e7a8259?utm_source=oschina-app)

[安装Kubernetes单Master节点](https://www.kuboard.cn/install/install-k8s.htm)