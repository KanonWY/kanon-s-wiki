# memory-ordering-at-compile-time

_**本文内容是对 Ref 中的翻译结合个人理解。**_

### 1、memory reorder rules(内存排序规则）
The cardinal rule of memory reordering, which is universally followed by compiler developers and CPU vendors, could be phrased as follows:  
Thou shalt not modify the behavior of a single-threaded program.  
编译器开发人员和 CPU 厂商所遵循的基本的内存排序规则为：  不得修改单线程程序的行为。

### 2、compiler instruction reordering （编译器指令重排）  
compiler is to convert human-readable soruce code into machine-readable code for the cpu.During this conversion, the compiler is free to take many liberties.
But In strong memory module cpu, the compiler might not reroder those operation.  
编译器最终重要的工作就是将源码翻译为机器码，在这个转换过程中有很大的自由操作空间，来优化代码的执行。但是在强内存模型中，编译器也许不会乱序指令。

### 3、compiler barrier（编译屏障）
开发人员可以通过一下的 5 个手段来阻塞编译的指令乱序。  
- 1、asm volatitle（手动填入汇编指令） 
we can use Explicit compiler barrier to prevent reordering of stores!
我们可以显示的在代码中写入编译屏障汇编来阻止编译器对该断代码进行内存乱序行为。  
example:
```c++
void foo()
{
  A = B + 1;
  asm volatitle("" ::: "memory");
  B = 0;
}
```
- 2、CPU fence （CPU 栅栏）
CPU fence instruction can also prevent reordering of stores!
- 3、cpp11 atomic operation （c++11 原子操作）
From C++11, atomic library standard, every non-relaxed atomic operation acts as a compiler barrier as well.
```c++
int Value { 0 };
std::atomic<int> at_int;

void SendInt(int x)
{
    Value = x;
    // pervented here!
    at_int.store(2, std::memory_order_release);
}
```
- 4、function which contain compiler barrier （包含编译屏障的函数）
function which contain compiler barrier , even the function is inline!
```c++
void doSomeThing(Foo* foo)
{
    foo->bar = 5;
    SendInt(233);            // prevents reordering of neighboring assignments
    foo->bar2 = foo->bar;
}
```
- 5、function call act as compiler barriers(not inline or pure)（普通非内联非纯函数）

通常的函数调用也能起到 编译屏障的作用。假设一个函数调用使用外部的库，编译器无法知道这个函数调用内部是否有依赖上下文中的数据，编译器也
无法知道上下文中的变量是否在该函数内部修改，因此为了遵守保存单线程结果一致的规则，它不得对外部调用函数的任何内存操作进行重新排序。
因此大多数情况下，编译器指令重排的情况比较难以见到（但是并非不存在).

### 4、现实中的编译器重排
从 ref 的文章描述的最后一章中提取出这么一些信息：  
现代编译器似乎减少了指令重排的操作，编译器可以生成更加高效的代码。

## Ref
[memory-ordering-at-compile-time](https://preshing.com/20120625/memory-ordering-at-compile-time/)