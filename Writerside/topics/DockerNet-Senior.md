# DockerNet_Senior

## 网络相关参数
### 1、服务器重启时生效的指令
指定容器挂载的网桥
```Bash
-b BRIDGE or --bridge=BRIDGE
```
指定 docker0 的掩码
```Bash
--bip=CIDR
```
Docker 服务器端接收命令的通道
```Bash
-H SOCKET... or --host=SOCKET...
```
是否支持容器之间进行通信
```Bash
--icc=true | false
```
发开转发功能
```Bash
--ip-forward=true | false
```
禁止 Docker 添加 iptables 规则
```Bash
--iptables=true | false
```
容器中的 MUT
```Bash
--mtu=BYTES
```
### 2、容器启动时生效的指令
下面指令既可以在启动服务的时候制定，也可以在容器启动的时候指定。  
DNS 服务器相关
```Bash
# 指定 DNS 服务器
--dns=IP_ADDRESS

# 指定 DNS 选项
--dns-ipt=""

# 制定 DNS 搜索域
--dns-search=DOMAIN
```

### 3、只能在容器启动的时候指定（与容器相关）
配置容器主机名
```Bash
-h HOSTNAME or --hostname=HOSTANME
```
制定容器内接口的 ip 地址
```Bash
-ip=""
```
添加到另一个容器的连接
```Bash
--link=CONTAINER_NAME:ALIAS
```
配置容器的桥接模式
```Bash
--net=bridge | none | container:NAME_OR_ID | host | user_defined_net
```
容器在网络中的别名
```Bash
--network-alias
```
映射容器端口到宿主机端口
```Bash
-p SPEC or --publish=SPEC

```
映射容器所有端口到宿主机端口
```Bash
-P or --publish-all=true|false
```
## 配置容器 DNS 和 主机名
Docker 服务启动后会默认启动一个内嵌的 DNS 服务，来动态解析一个网络中的容器主机名和地址，
如果无法解析，而可以通过容器内的 DNS 相关配置进行解析。
### 相关配置文件
```Bash
/etc/resolv.conf
/etc/hostname
/etc/hosts
```
在容器中使用 mount 指令可以看到挂在了三个 DNS 相关的文件。
```Bash
mount
overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/VH5VSIRHZUY4VRJYW4VT5C4LNH:/var/lib/docker/overlay2/l/LAVZCIVQTVC6UZ3L6FBZQ26MDJ,upperdir=/var/lib/docker/overlay2/63bb9a11007daf33b19aaa219b0004ab42f74e0bf96afce37204ee8f468674c1/diff,workdir=/var/lib/docker/overlay2/63bb9a11007daf33b19aaa219b0004ab42f74e0bf96afce37204ee8f468674c1/work)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
tmpfs on /sys/fs/cgroup type tmpfs (rw,nosuid,nodev,noexec,relatime,mode=755,inode64)
cgroup on /sys/fs/cgroup/systemd type cgroup (ro,nosuid,nodev,noexec,relatime,xattr,name=systemd)
cgroup on /sys/fs/cgroup/devices type cgroup (ro,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (ro,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (ro,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/rdma type cgroup (ro,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (ro,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/misc type cgroup (ro,nosuid,nodev,noexec,relatime,misc)
cgroup on /sys/fs/cgroup/blkio type cgroup (ro,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/freezer type cgroup (ro,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/memory type cgroup (ro,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/pids type cgroup (ro,nosuid,nodev,noexec,relatime,pids)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k,inode64)
/dev/nvme1n1p2 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro)
/dev/nvme1n1p2 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro)
/dev/nvme1n1p2 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
proc on /proc/bus type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/fs type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/irq type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sys type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sysrq-trigger type proc (ro,nosuid,nodev,noexec,relatime)
tmpfs on /proc/asound type tmpfs (ro,relatime,inode64)
tmpfs on /proc/acpi type tmpfs (ro,relatime,inode64)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
tmpfs on /proc/keys type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
tmpfs on /proc/timer_list type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
tmpfs on /proc/scsi type tmpfs (ro,relatime,inode64)
tmpfs on /sys/firmware type tmpfs (ro,relatime,inode64)
```
Docker 容器启动的时候，会从宿主机上赋值 /etc/resolv.conf 文件，并且删除无法连接到的 DNS 服务器。
/etc/host 中仅仅记录了容器自身的地址和名称，而 /etc/hostname中则记录了容器的主机名。  
容器内修改这三个配置文件在容器终止后失效，并且不会被 _**docker commit**_ 提交。
```Bash
-h HOSTNAME
设置容器的主机名，会被写到容器内 /etc/hostname 和 /etc/hosts 中，
但是容器外部无法看到。

--link=CONATINAER_NAME:ALIAS
记录其他容器的主机名。在创建容器的时候，添加一个所连接的容器主机名字
到容器内的 /etc/hosts 文件中。
这样，新建容器可以使用主机名字与所连接容器通信

--dns=IP_ADDRES
指定 DNS 服务器，添加 DNS 服务器到容器内部的 /etc/resolv.conf 中，
容器会用指定的 DNS 服务器来解析不在 /etc/hosts 中的主机名
```
## 容器访问控制
Docker 容器的访问控制通过 Linux 上的 iptables 防火墙软件来进行管理是和实现。
### 容器访问外部网络
默认情况下容器可以访问到宿主机的本地网络，因为宿主机的本地网络与容器网络使用 docker0 网桥连接。但是容器想要通过宿主机器访问外部网络，
需要宿主机进行辅助转发。默认情况下，Docker 服务器启动时候会开启 --ip-forward=true,自动配置宿主机的转发规则
```Bash
# 查看是否开启转发
sudo sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
```
### 容器之间的互相访问
容器之间访问需要两个条件
- 网络拓扑连通：默认所有的容器都在一个网桥上，意味着默认是联通的
- 本地系统的防火墙是否运行通过，取决于防火墙的规则（大部分是禁止的）
