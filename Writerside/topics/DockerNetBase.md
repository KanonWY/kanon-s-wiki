# DockerNetBase

## Linux 网络虚拟化
Docker 本地网络利用了 Linux 网络命名空间以及虚拟网络设置（veth pair）,实现了本地
容器间的通信以及，本地容器与宿主机之间的通信。
### 基本原理
要实现网络通信，必须需要一个网络接口与外界通信，并且可以发送数据包；此外不同子网之间通信
还需要路由机制。  
Docker 中的网络接口默认都是虚拟接口。虚拟接口具有转发效率高的优势，因为在linux内核中
可以使用数据复制来进行虚拟接口之间的数据转发，也即发送缓存中的数据包将被直接复制到接收接口
的缓冲中。Docker 在本地和容器内部分别创建了一个虚拟接口来通信。
### 基本创建过程
- 1、创建一对虚拟接口，分别放到主机和新容器的命名空间中
- 2、本地主机一端的虚拟接口连接到默认的 docker0 网桥或者制定网桥上，并且具有一个 veth
开头的唯一名字
- 3、容器一段将虚拟接口放到新创建的容器中，修改名字为 eth0, 该接口只在容器的命名空间可见。
- 4、网桥从可用地址空间中获取一个空闲地址分配给容器的 eth0, 并配置默认路由网关为 docker0
网卡的内部接口的 ip 地址。

完成上述接口之后，容器可以使用它能看到的 eth0 虚拟网卡来接口其他容器和外部网络。

在 docker 容器启动改的时候 **--net=xxx** 选项可以制定容器的网络配置。
#### --net=bridge
默认值，使用 docker0 为容器创建网络栈 
#### --net=non: 
让 docker 将新的容器放到隔离网络栈中，但是不进行网络配置，让用户自行配置
#### --net=container:NAME_OR_ID
让 docker 将新建的容器的进程放到一个已经存在容器网络栈中，新容器有自己的文件系统，进程列表
和资源限制，但是会和已经存在的容器共享 IP 地址和断后等网络资源，两者进程可以直接通过 lo 环回接 通信。
#### --net=host
让 docker 不要将容器的网络放到隔离的命名空间中，也即不要容器化容器内部的网络。此时容器使用的
是本地主机的网络，它拥有完全的本地主机接口访问权限。容器进程跟主机其他 root 进程一样可以打开低范围的端口，
可以访问本地网络服务，如果进一步添加 --privileged=true 参数，容器设置可以配置主机网络栈。
#### --net=user_defined_network
用户自行使用 network 命令创建一个网络，然后将容器连接到自己创建的网络中去。

### 手动配置 docker 本地网络
```Bash
# 运行容器
docker run -it --rm --net=none ubuntu20:18.04 /bin/bash

# 获取容器进程id {pid}
docker inspect -f '{{.State.Pid}}' {container_id}

# 为容器进程创建网络命名空间
sudo mkdir -p /var/run/netns
sudo ln -s /protc/$pid/ns/net /var/run/netns/$pid

# 检查网桥网卡的 IP 和子网掩码信息
ip addr show docker0

4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:29:ee:5c:44 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:29ff:feee:5c44/64 scope link 
       valid_lft forever preferred_lft forever

# 创建 veth pair 虚拟网口 A 、B 
sudo ip link add A type veth peer name B

# 绑定 A 接口到 docker0 网桥
sudo brctl addif docker0 A

# 启动 A 接口
sudo ip link set A up

# 将接口 B 放到容器的网络命名空间中（需要容器的PID)
sudo ip link set B netns $pid

# 使用 ip netns exec 命令在指定网络命名空间中进行配置

# 1、将在容器网络命名空间中的 B 接口命名为 eth0
sudo ip netns exec $pid ip link set dev B name eth0

# 2、启动 eth0
sudo ip netns exec $pid ip link set eth0 up

# 3、配置一个可用的 IP
sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0

# 4、设置默认网关
sudo ip netns exec $pid ip route add default via 172.117.42.1
```