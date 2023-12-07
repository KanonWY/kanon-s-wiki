# pythonExample

## Send
```Python
#! /bin/python3
import socket
import struct

# 设置组播地址和端口号
multicast_group = '239.2.3.4'  # 请替换为实际的组播地址
port = 12345  # 请替换为实际的端口号

# 创建组播套接字
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 绑定到指定的端口
sock.bind(('', 0))

# 设置TTL（生存时间）
ttl = struct.pack('b', 2)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, ttl)

# 发送组播消息
message = 'Your multicast message'
sock.sendto(message.encode(), (multicast_group, port))

# 关闭套接字
sock.close()
```

## Rec
```Python
#！/bin/python3
import socket
import struct

# 设置组播地址和端口号
multicast_group = '239.2.3.4'  # 请替换为实际的组播地址
port = 12345  # 替换为实际的端口号

# 创建UDP套接字
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 绑定到接收所有数据的端口
sock.bind(('', port))

# 告诉内核将数据包发送到多播组
group = socket.inet_aton(multicast_group)
mreq = struct.pack('4sL', group, socket.INADDR_ANY)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

# 接收组播消息
while True:
    data, address = sock.recvfrom(1024)
    print(f'Received {len(data)} bytes from {address}: {data.decode()}')

# 关闭套接字
sock.close()
```