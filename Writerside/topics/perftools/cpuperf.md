# googleperf-cpu
## 1、cpu_tools
谷歌性能工具可以查看程序运行的CPU消耗。
这段代码是 C 语言中用于配置 ProfilerOptions 的结构体定义。此结构体用于在启动性能剖析器时使用的选项配置。下面是一些字段的说明：

1. `filter_in_thread`：这是一个函数指针，用于设置一个过滤器函数。在每次收到性能剖析器的采样时，剖析器将调用 `filter_in_thread` 函数，并传入 `filter_in_thread_arg` 作为参数。如果 `filter_in_thread` 返回非零值，表示该采样应该包含在剖析结果中。需要注意的是，`filter_in_thread` 函数在信号处理器中运行，因此必须是异步信号安全的函数。
2. `filter_in_thread_arg`：这是传递给 `filter_in_thread` 函数的参数。

通过设置这些 ProfilerOptions 结构体的字段，开发人员可以自定义剖析器的行为以满足特定的需求。例如，可以使用 `filter_in_thread` 函数实现基于线程的过滤逻辑，通过检查线程特定的数据来决定是否包含特定线程的采样数据。

需要注意的是，这个结构体是为了在 C 代码中使用，因此没有提供构造函数来初始化它。开发人员可以使用 memset 函数将其填充为零，然后根据需要填充字段的值。

## 2. 基于 perf  实现基于线程过滤

## 3. 使用信号控制 perf cpu

## Ref
[cpuprofile DOC](https://gperftools.github.io/gperftools/cpuprofile.html)