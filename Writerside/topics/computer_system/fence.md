# Memory Fence

### 1. 概念
由于 CPU 流水线执行，以及 StoreBuf 和 Invalidate queue 对缓存一致性的优化，导致 CPU 指令会乱序执行，并且共享变量的可见性无法被保证。


### 2. Java 中的屏障

## Ref
[liwuzhi](http://liwuzhi.art/?p=877)  
[atomic-variable-mutex-and-memory-barrier](https://www.0xffffff.org/2017/02/21/40-atomic-variable-mutex-and-memory-barrier/)  
[ref1](https://zhuanlan.zhihu.com/p/43526907)  
[ref2](https://zhuanlan.zhihu.com/p/35386457)  
[ref3](https://zhuanlan.zhihu.com/p/41872203)  
[ref4](https://sf-zhou.github.io/programming/memory_barrier.html)  
[内存屏障](https://zhuanlan.zhihu.com/p/606658179)