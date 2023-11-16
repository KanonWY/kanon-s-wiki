# 冰羚
冰羚中间件：
也是采用共享内存进行零拷贝。目前看来不太一样的是，这个使用使用一个节点来管理所有的内存池。

## 编码准则
- 禁止使用堆内存  
- 只使用STL 标准模版  
- 禁止未定义行为  
- 禁止使用异常  

## Ref
[iceoryx 简单使用](https://blog.csdn.net/weixin_35762621/article/details/125597628?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-125597628-blog-131436194.235%5Ev38%5Epc_relevant_default_base&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-125597628-blog-131436194.235%5Ev38%5Epc_relevant_default_base&utm_relevant_index=1)  
[iceoryx arch](https://iceoryx.io/v2.0.3/concepts/architecture/)