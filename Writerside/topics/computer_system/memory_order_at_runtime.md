# Memory-ordering-at-runtime

### 1、运行时乱序的产生
由于 CPU 采用流水线程执行，以及指令之间的依赖关系，会导致实际执行的指令与编译后二进制的顺序不同。

### 2、乱序的后果
乱序带来的后果在单处理和 SMP 处理器上都有一定的影响。
#### 单处理器的影响
尽管前文提到，合理的乱序是在保证不影响单线程逻辑的前提下，这意味着他只能识别简单的上下文因果关系。对于一些隐藏的因果关系甚至多线程关系无能为力。
特殊情况下：程序逻辑的正确性依赖于内存访问顺序，这时候内存乱序访问会带来逻辑上的错误
这里举一个多线程视角下的例子：
```c++
struct Object
{
    char* data;
    bool ready{false};
};

Object g_obj;

// Thread1
{
    g_obj->data = malloc(200);      //(1)
    g_obj->ready = true;              //(2)
}

// Thread2
{
    if(g_obj->ready)
    {
        do_work(obj->data);         // ERROR
    }
}
```
线程 1 执行时候，有可能将 (1) 和 （2）乱序执行，执行完 （2） 后，由于系统任务调用切换到了线程 2 上，这时候 data 字段还没设置，就会出现问题。
因为对于 CPU 而言，语句 （1） 和 （2） 没有因果关系，是可能替换的。
可以使用写内存屏障，保证在 g_obj->ready = true 设置成功的时候，g_obj->data 一定已经分配了内存了。

#### SMP 上的影响
上面讨论了单处理上的指令乱序问题，多处理器下处理上述问题外，还需要解决因为每一个 CPU 上都有自己的多级缓存引起的数据可见性问题。
处理器对内存的写操作不是直接在主存上生效的，而是先经过自身的缓存，然后同步到主存上，另一个处理器读取的时候，从主存上是有可能读取到旧数据的。
```c++
<CPU-a>                                <CPU-b>
g_obj->data = xxx;              
wmb();
g_obj->ready = true;                   if(g_obj->ready)
                                           do_work(g_obj->data);
```
上面提到，使用内存屏障保证两次赋值之间不乱序，从而使得 ready 标志为 true 的时候，data 一定是有效的。但是在多处理器的情况下，这样还是存在问题。
data 和 ready 值可能以相反的顺序更新到 CPU-b 的缓存中，从而被 CPU-b 读取到。  
（wmb() 还能保证屏障之前的cache更新消息先于屏障之后的消息被发出。？）
为什么会以相反的次序更新到 CPU-b ？ 
CPU-b 可能由于缓存列的繁忙程度，导致 data 的更新晚于 ready 的更新。因此 CPU-b 上需要使用读屏障，保证两个 cache 单元的同步不乱序。
因此在 SMP 下，内存屏障必须配对使用才能生效。
```c++
<CPU-a>                                <CPU-b>
g_obj->data = xxx;              
wmb();  // 写屏障
g_obj->ready = true;                   if(g_obj->ready)     // 到这里说明 ready 更新了，但是 data 可能还没有更新
                                       {
                                           rmb();           // 读屏障，有了读屏障可以保证取到最新内存数据了。
                                           do_work(g_obj->data);                                       
                                       }
```
在 SMP 下，内存屏障保证的是 “**一个 CPU 的多个操作的顺序被一个 CPU 所观测到的顺序是一致的！**”,但是并不保证两个 "**两个 CPU 的操作顺序**"。

```c++
CPU-a                     CPU-b

a=5                       rmb();
wmb();                    i = a;
```
i 并不一定等于 5，CPU 上两个指令执行的先后没有关联，程序没有要求谁必须先执行，谁必须后执行。

### 内存屏障

### 写屏障
在指令后插入Store Barrier，能让写入缓存中的最新数据更新写入主内存，让其他线程可见。即写屏障之前的写操作一定会在写屏障之后完成，让其它线程可见。

### some detail in hardware
http://www.rdrop.com/users/paulmck/scalability/paper/whymb.2010.07.23a.pdf



## Ref
[preshing's Blog](https://preshing.com/20120612/an-introduction-to-lock-free-programming/)  
[CPP并发编程作者的博客 ](https://www.justsoftwaresolutions.co.uk/blog/)   
[Dmitry Vyukov's Blog](https://www.1024cores.net/)   
[Charles Bloom’s Blog](https://cbloomrants.blogspot.com/2012/06/06-12-12-another-threading-post-index.html)  

[写屏障](https://blog.csdn.net/lz710117239/article/details/123457011)    
[读屏障](https://blog.csdn.net/lz710117239/article/details/123460027)  
[知乎内存屏障](https://zhuanlan.zhihu.com/p/491567798)  
[罗 知乎](https://zhuanlan.zhihu.com/p/45808885)  