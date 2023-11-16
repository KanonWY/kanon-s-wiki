# MESI

### Write through And Write Back

#### Write through
检查数据是否在 Cache 中，如果不在直接写回内存中，如果存在，先写到 Cache 中，再写到内存中。最后数据都要写到内存中。
#### Write Back
目的: 减少把数据写回到内存中的频率。  
新的数据仅仅被写入到 Cache Block 中，只有修改过的 Cache Block 被替换，才真正写到内存中。

### State
- M: 该 Cache Line 为该 CPU 核独有，而且尚未写回到主存中
- E: 该 Cache Line 为该 CPU 核独有，且与主存一致
- S: 该 Cache Line 被多 CPU 核共享，且与主存一致
- I: 该 Cache Line 失效, 不能读取该 Cache Line 数据

多个 CPU 核心使用使用高速通道进行通信。

### Message Type
- 读消息
请求消息，用于通知其他处理器、主内存，
- 读消息响应
- 使无效消息
- 使无效消息的响应

### State Change
Cache Line 状态转换图
<img alt="MESI 状态转换" height="550" src="MESI.png" width="450"/>
**典型的例子**：CPU-a 修改共享变量 Flag, CPU-b 获取 a 过程。

```C++
// global
MsgClass *msg = nullptr;
std::atomic<bool> flags {false};

// Thread A （CPU-a）
msg = new MsgClass;
flags.store(true, std::memory_order_relaxed);

// Thread B （CPU-b）
while(!flag.load(std::memory_order_relaxed));
assert(msg != nullptr)
```
假设初始状态下， CPU-a 和 CPU-b 的中的对于 Flag 的 Cache Line 状态都是 S 状态； 
CPU-a 修改 Flag 的内容，会直接修改 Cache Line 中的内容，将该 Cache Line 变为 M, 
同时发送使用无效消息，CPU-b 核心收到无效消息，将本地该 Cache Line 标记为 I 状态。

CPU-b 读取 Flag 消息，由于此时的 Cache Line 状态为 I, 并且其他核心有这个份数据，并且状态为 M,则将数据更新到内存，并且
本地核心中的 Cache Line 从内存中读取数据，然后将 CPU-b 和 CPU-a 中的 Flag 对应的 Cache Line 修改为 S 状态。

假设 CPU0-3 四个 CPU 中都有一个 Cache Line，包含共享变量X, 并且状态都是 S.
此时 CPU0 要对 X 进行写操作，这时候通过通信机制（Snoop 机制）发出 Invalidate 消息，CPU1-3 中的缓存行会置为 I 状态，然后给 CPU0 发送响应（Invalidate Ack）,
收到全部的响应之后， CPU0 会完成对于变量 X 的写操作，更新 CPU0 内缓存行的状态为 M, 但是不会同步到内存中。  
若干个指令周期之后， CPU1 想要对变量 X 执行读操作，此时它发现本地缓存行时 I 状态，就会触发 CPU0 将缓存写回到内存中，然后 CPU1
再从主存中同步最新的值，然后 CPU0 和 CPU1 的该缓存行被设置为 S.

### Store Buffer
前面说到的 CPU0 发出 Invalidate 消息之后，其实并不会等到其他的 CPU 都将状态设置为 I 然后发送 Ack 之后再去写操作，更新自己 Cache Line 状态。
这里为了消除等待响应的同步行为，引入了 Store Buffer 技术，也即 CPU0 执行写操作的时候直接写到 Store Buffer 中然后去做别的操作，等到所有的其他的
CPU 都将状态置为 I 之后，才将 Store Buffer 中的数据写入到 Cache Line 中。
### Invalidate Queue
上面写端的 CPU 使用 Store Buffer 消除了同步等待其他 CPU 缓存行失效标志设置返回 ack 的过程。 而其他都到 Invalidate 消息的 CPU 也不会真的
把 Cache Line 的标志为设置为 I 之后才响应 ACK, 而是将它写入到一个 Invalidate Queue 中，然后就发送 Ack 了。 后续 CPU 会异步扫描 Invalidate 
Queue, 批量设置为 I 状态。  
和 Store Buffer 不同的是，CPU0 后续读取变量 X 的时候，会先查询 Store Buffer, 然后查询缓存。 而 CPU1 则不会扫描 invalidate Queue,因此可能存在脏读。

## Ref
[MESI details](https://www.cnblogs.com/jiagoujishu/p/13799459.html)   
[MESI](https://zhuanlan.zhihu.com/p/33445834)  
[CPU 缓存一致性-MESI-brpc](https://zhuanlan.zhihu.com/p/351550104)  
[CPU 缓存一致性](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247486479&idx=1&sn=433a551c37a445d068ffbf8ac85f0346&chksm=f98e48a5cef9c1b3fadb691fee5ebe99eb29d83fd448595239ac8a2f755fa75cacaf8e4e8576&scene=21#wechat_redirect)  
