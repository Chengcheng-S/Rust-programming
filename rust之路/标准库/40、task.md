# std::task

处理异步任务的类型和triat

### Macros

ready			提取Poll <T>的成功类型

### Enums

Poll              指示值是否可用，或者是否已安排当前任务接收唤醒。

### Traits

Wake  实验：在执行程序上唤醒任务的实现

### Structs

| Context        | 异步任务的上下文                                             |
| -------------- | ------------------------------------------------------------ |
| RawWaker       | RawWaker允许任务执行器的实现者创建提供自定义唤醒行为的Waker。 |
| RawWakerVTable | 一个虚拟函数指针表（vtable），用于指定RawWaker的行为。       |
| Waker          | Waker是通过通知执行者准备运行来唤醒任务的句柄。              |



