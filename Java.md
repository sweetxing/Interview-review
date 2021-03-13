## 一、IO

### 1.1 概述

- **同步与异步**

消息通信机制，是一种通信状态。立即返回（无结果）or主动等待结果再返回。

- **阻塞与非阻塞**

关注的是我是否会因为等待结果而让出cpu。

### 1.2 IO的字符流和字节流

### 1.3 NIO（同步、非阻塞）

**三个重要组成部分：** Channel、Buffer、Selector

- **Channel**

双工通信（区别于流的单向性，通道是双向的）、可以异步读写、必须通过buffer读写

- **Buffer**

四步骤：write to buffer -> flip -> read from buffer -> clear

- **Selector**

一个selector管理多个channel，以减少线程间的切换

### 1.4 AIO（异步、非阻塞）

基于事件和回调机制实现的。（貌似应用不广泛）



