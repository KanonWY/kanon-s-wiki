# Acquire and Release

### definitions
Acquire semantics is a property that can only apply to operations that read from shared memory, whether they are read-modify-write operation or plain loads.
The opration is then considered a read-acquire. 



### 不同的处理器有不同的内存模型
PowerPC 和 ARM 处理器可以更改内存存储相对于指令本身的顺序，但通常情况下，Intel 和 AMD 的 x86/64 系列处理器不会这样做。  
X86/64 平台特点：
 it’s that at the x86/64 instruction level, every load from memory comes with acquire semantics, and every store to memory provides release semantics – 
 at least for non-SSE instructions and non-write-combined memory. 

 
