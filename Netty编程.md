# Netty编程

## NIO基础

non-Blocking io 非阻塞 IO

### 三大组件

#### Channel

类似stream，读写数据的双向通道

#### Buffer

可以从Channel将数据读入buffer，用来缓冲读写数据

#### Selector

过去的模型

1）多线程设计

![image-20211205154617515](C:\Users\wy\Desktop\笔记\images\image-20211205154617515.png)

缺点：内存占用高，上下文切换成本高，只适合连接数少的场景

