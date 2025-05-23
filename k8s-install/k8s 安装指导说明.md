## Step 1 ： 首先在每台机器上设置hostname
#### 1.1 : 修改 hostname master.mistra.com 或者 node1.mistra.com
`hostnamectl set-hostname xxxxxxx`
#### 1.2 : 查看修改结果
`hostnamectl status`
#### 1.3 : 设置 hostname 解析
`echo "127.0.0.1   $(hostname)" >> /etc/hosts`

## Step 2 ：机器检查
所有节点必须保证以下条件
- 任意节点 centos 版本为 7.6 , 7.7 , 7.8 或者 centos stream 8
- 任意节点 CPU 内核数量大于等于 2，且内存大于等于 4G
- 任意节点 hostname 不是 localhost，且不包含下划线、小数点、大写字母
- 任意节点都有固定的内网 IP 地址
- 任意节点都只有一个网卡，如果有特殊目的，我可以在完成 K8S 安装后再增加新的网卡
- 任意节点上 Kubelet使用的 IP 地址 可互通（无需 NAT 映射即可相互访问），且没有防火墙、安全组隔离
- 任意节点不会直接使用 docker run 或 docker-compose 运行容器


## Step 3 ：base-install.sh 所有节点基础安装
把 base-install.sh 拷贝到服务器 
- 执行 这是阿里云镜像地址：
  `export REGISTRY_MIRROR=https://registry.cn-shanghai.aliyuncs.com`
- 执行 `./base_install.sh 1.21.5`

## Step 4 : 初始化 Master 节点 （只在 Master 节点执行）
### 4.1 ：设置变量
```
# 只在 master 节点执行
# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP） 
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=x.x.x.x
# 替换 apiserver.mistra.com 为 您想要的 dnsName 
export APISERVER_NAME=apiserver.mistra.com
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
export POD_SUBNET=10.100.0.1/16
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
```

### 4.2 ：执行 `./install_master.sh 1.21.5`

### 4.3 : 检查执行结果
```
# 只在 master 节点执行
# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态
watch kubectl get pod -n kube-system -o wide

# 查看 master 节点初始化结果
kubectl get nodes -o wide 
```
### 4.4 : 在Master节点上安装 Flannel 网络插件
flannel-v0.14.0.yaml

```
export POD_SUBNET=10.100.0.0/16

sed -i "s#10.244.0.0/16#${POD_SUBNET}#" flannel-v0.14.0.yaml

kubectl apply -f ./flannel-v0.14.0.yaml
```

## Step 5 ：初始化 Worker 节点
### 5.1 ： 首先在 Master 节点上执行以下命令
```
# 只在 master 节点执行
kubeadm token create --print-join-command

```
可获取kubeadm join 命令及参数，如下所示
```
# kubeadm token create 命令的输出，形如：
kubeadm join apiserver.mistra.com:6443 --token o5vmo9.bazxuhkyew9rajvi     --discovery-token-ca-cert-hash sha256:956583e510265cb6ec4bd5f11f36a05917e822aa7e3fbf950bce0e6d732ad956 

```
>该 token 的有效时间为 2 个小时，2小时内，您可以使用此 token 初始化任意数量的 worker 节点。

### 5.2 : 初始化 worker （只在worker 节点执行）

```
# 只在 worker 节点执行
# 替换 x.x.x.x 为 master 节点的内网 IP
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=x.x.x.x
# 替换 apiserver.mistra.com 为 您想要的 dnsName 
export APISERVER_NAME=apiserver.mistra.com
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
```

### 5.3 : 执行 Master 节点上 token 信息加入集群
```
# 替换为 master 节点上 kubeadm token create 命令的输出
kubeadm join apiserver.mistra.com:6443 --token o5vmo9.bazxuhkyew9rajvi     --discovery-token-ca-cert-hash sha256:956583e510265cb6ec4bd5f11f36a05917e822aa7e3fbf950bce0e6d732ad956 

```
           
### 5.4 :检查初始化结果
在 master 节点上执行（只在Master上）
`kubectl get nodes -o wide`