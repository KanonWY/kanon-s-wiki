# Memory Fence

https://www.0xffffff.org/2017/02/21/40-atomic-variable-mutex-and-memory-barrier/

https://zhuanlan.zhihu.com/p/43526907

https://zhuanlan.zhihu.com/p/35386457

https://zhuanlan.zhihu.com/p/41872203

https://sf-zhou.github.io/programming/memory_barrier.html


### 内存屏障原理：  
http://liwuzhi.art/?p=877 该文章简单介绍了内存屏障存在的意义。  
代码书写的顺序和实际执行的顺序也许不一致：  
- 编译器优化，编译器尝试优化出更加高效的代码。
- cpu 并发执行，cpu会同时加载执行多条指令。
- cpu 缓存，多级缓存导致出现可见性问题。
#### 1、指令优化
优化的本质是对操作的替换和指令重新编排。  
操作的替换：原本从内存中读取，改为从寄存器中，或者缓存中。  
指令的重新编排：替换指令的执行顺序，从而加快程序的整体执行效率。  
指令优化的原则：  
优化后代码执行的结果与未优化后的结果一致。优化后的代码执行不会改变 sequenced-before 关系。
编译器只需要保证在单线程的条件下，最终一致性即可。它只知道当前线程中的数据读写与数据依赖关系，对于共享的变量无感知，它并不知道哪些变量会被共享，哪些变量会被修改。这些需要开发人员去保证。  

#### 2、阻止优化
为什么需要阻塞优化？在多线程的环境下，由于编译器和 CPU 硬件的优化会导致多线程的消息共享出现问题。需要一些指令控制编译器和CPU执行的优化，来保证多线程程序的执行正确性。  
大多数硬件都提供了一些指令来控制指令执行的次序。  
比如： MM_BOTTOM指令，该指令之前的指令都必须执行。

```c++
int a = 300;    //A
MM_BOTTOM
int b = 400;    //B
```
这样 MM_BOTTOM 之前的指令都必须被执行，但是这样并没有限制 B 这些指令，B 完全可以跑到 MM_BOTTOM 前面执行：
```c++
int a = 300;  // A
int b = 400;  // B
MM_BOTTOM
```
或者跑到任意位置：
```c++
int b = 400;  // B
int a = 300;  // A
MM_BOTTOM
```
这个时候就需要另一条指令防止： B 跑到前面去: MM_TOP

```c++
int a = 300;
MM_BOTTOM （之前的指令必须执行）
MM_TOP     (之后的指令都不能执行）
int b = 400;

```



