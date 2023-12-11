# netfilter_iptables

## 基本概念

### 1、IP 数据包的流转
#### 到本机某进程的 IP 数据报
网口 -> PREROUTING -> INPUT -> 应用层处理
#### 本机转发的报文
网口 -> PREROUTING -> FORWARD -> POSTROUTING
### 本机某一个进程发出的报文
应用层 -> OUTPUT -> POSTROUTING

### chain


## Ref
[深入理解 netfilter 和 iptables](https://zhuanlan.zhihu.com/p/545050331)  
[iptables详解](https://www.zsythink.net/archives/tag/iptables/)  
[iptables](https://zhuanlan.zhihu.com/p/429329901)  
[iptables最常用的规则示例](https://zhuanlan.zhihu.com/p/665621141)  
[iptables example](http://cn.linux.vbird.org/linux_server/0250simple_firewall_3.php)
