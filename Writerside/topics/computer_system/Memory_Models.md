# Memory Models

Each physical device have their different hardware memory model. A hardware memory model tell us what kind of memory ordering to expect at runtime relative 
to an assembly.  
物理设备有它们自己的硬件内存模型，不同的内存模型体现了它们在运行时候对指令乱序表现。

### weak Memory models
In the weakest memory model, it's possible to experience all four types of memory reordering. Any load or store operation can effectively be recordered with
any other load or store operation, as long as it would never modify the behavior of a single thread. In reality, the reordering may be due to compiler reordering
or memory reordering on the processor itself!  

