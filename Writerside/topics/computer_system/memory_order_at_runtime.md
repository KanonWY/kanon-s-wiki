# Memory order at runtime

#### LoadLoad fence
LoadLoad fence 能够有效的阻止在 内存屏障之前的 load 操作，与在内存屏障之后的 load 操作重排。  
现代处理器中，可能会对读取加载操作进行重排序。LoadLoad 屏障会阻止在它之前的 Load 操作与在它之后 的 Load 操作进行重排序。这意味着在 LoadLoad
屏障之前的读取操作将在该内存屏障之前完成，位于LoadLoad 之后的 Load 操作将在该屏障之后完成。

#### StoreStore fence
StoreStore fence 能够有效阻塞在内存屏障之前的 store 操作和之后的 store 操作重排。

#### LoadStore fence

#### StoreLoad fence


#### some detail in hardware
http://www.rdrop.com/users/paulmck/scalability/paper/whymb.2010.07.23a.pdf



## Ref
[preshing's Blog](https://preshing.com/20120612/an-introduction-to-lock-free-programming/)  
[CPP并发编程作者的博客 ](https://www.justsoftwaresolutions.co.uk/blog/)   
[Dmitry Vyukov's Blog](https://www.1024cores.net/)   
[Charles Bloom’s Blog](https://cbloomrants.blogspot.com/2012/06/06-12-12-another-threading-post-index.html)  