## 基本介绍

Netty 是一个异步事件驱动的网络应用程序框架，用于快速开发可维护、高性能的网络服务器和客户端

Netty 官网：https://netty.io/

Netty 的对 JDK 自带的 NIO 的 API 进行封装，解决上述问题，主要特点有：

- 设计优雅，适用于各种传输类型的统一 API， 阻塞和非阻塞 Socket 基于灵活且可扩展的事件模型
- 使用方便，详细记录的 Javadoc、用户指南和示例，没有其他依赖项
- 高性能，吞吐量更高，延迟更低，减少资源消耗，最小化不必要的内存复制
- 安全，完整的 SSL/TLS 和 StartTLS 支持

Netty 的功能特性：

* 传输服务：支持 BIO 和 NIO
* 容器集成：支持 OSGI、JBossMC、Spring、Guice 容器
* 协议支持：HTTP、Protobuf、二进制、文本、WebSocket 等一系列协议都支持，也支持通过实行编码解码逻辑来实现自定义协议
* Core 核心：可扩展事件模型、通用通信 API、支持零拷贝的 ByteBuf 缓冲对象

![输入图片说明](https://foruda.gitee.com/images/1679449905811977811/064535c6_8616658.png "屏幕截图")



## 线程模型

### 阻塞模型

传统阻塞型 I/O 模式，每个连接都需要独立的线程完成数据的输入，业务处理，数据返回

<img src="https://seazean.oss-cn-beijing.aliyuncs.com/img/Frame/Netty-传统阻塞IO服务模型.png" style="zoom:50%;" />

模型缺点：

- 当并发数较大时，需要创建大量线程来处理连接，系统资源占用较大
- 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在 read 操作上，造成线程资源浪费



***

#### 单R单线程

Reactor 对象通过 select 监控客户端请求事件，收到事件后通过 dispatch 进行分发：

* 如果是建立连接请求事件，则由 Acceptor 通过 accept 处理连接请求，然后创建一个 Handler 对象处理连接完成后的后续业务处理

* 如果不是建立连接事件，则 Reactor 会分发给连接对应的 Handler 来响应，Handler 会完成 read、业务处理、send 的完整流程

  说明：**Handler 和 Acceptor 属于同一个线程**

<img src="https://seazean.oss-cn-beijing.aliyuncs.com/img/Frame/Netty-单Reactor单线程.png" style="zoom:50%;" />

模型优点：模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成

模型缺点：

* 性能问题：只有一个线程，无法发挥多核 CPU 的性能，Handler 在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
* 可靠性问题：线程意外跑飞，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障

使用场景：客户端的数量有限，业务处理非常快速，比如 Redis，业务处理的时间复杂度 O(1)



***

#### 单R多线程

执行流程通同单 Reactor 单线程，不同的是：

* Handler 只负责响应事件，不做具体业务处理，通过 read 读取数据后，会分发给后面的 Worker 线程池进行业务处理

* Worker 线程池会分配独立的线程完成真正的业务处理，将响应结果发给 Handler 进行处理，最后由 Handler 收到响应结果后通过 send 将响应结果返回给 Client

<img src="https://seazean.oss-cn-beijing.aliyuncs.com/img/Frame/Netty-单Reactor多线程.png" style="zoom:50%;" />

模型优点：可以充分利用多核 CPU 的处理能力

模型缺点：

* 多线程数据共享和访问比较复杂
* Reactor 承担所有事件的监听和响应，在单线程中运行，高并发场景下容易成为性能瓶颈


***



#### 主从模型

采用多个 Reactor ，执行流程：

* Reactor 主线程 MainReactor 通过 select **监控建立连接事件**，收到事件后通过 Acceptor 接收，处理建立连接事件，处理完成后 MainReactor 会将连接分配给 Reactor 子线程的 SubReactor（有多个）处理

* SubReactor 将连接加入连接队列进行监听其他事件，并创建一个 Handler 用于处理该连接的事件，当有新的事件发生时，SubReactor 会调用连接对应的 Handler 进行响应

* Handler 通过 read 读取数据后，会分发给 Worker 线程池进行业务处理

* Worker 线程池会分配独立的线程完成真正的业务处理，将响应结果发给 Handler 进行处理，最后由 Handler 收到响应结果后通过 send 将响应结果返回给 Client

<img src="https://foruda.gitee.com/images/1679451451236013041/4ede8286_8616658.png" style="zoom:50%;" />

模型优点

- **父线程与子线程**的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理
- 父线程与子线程的数据交互简单，Reactor 主线程只需要把新连接传给子线程，子线程无需返回数据

使用场景：Nginx 主从 Reactor 多进程模型，Memcached 主从多线程，Netty 主从多线程模型的支持



***

### Proactor

Reactor 模式中，Reactor 等待某个事件的操作状态发生变化（文件描述符可读写，socket 可读写），然后把事件传递给事先注册的 Handler 来做实际的读写操作，其中的读写操作都需要应用程序同步操作，所以 **Reactor 是非阻塞同步网络模型（NIO）**

把 I/O 操作改为异步，交给操作系统来完成就能进一步提升性能，这就是异步网络模型 Proactor（AIO）：

![输入图片说明](https://foruda.gitee.com/images/1679451394410244164/8ab3d7fa_8616658.png "屏幕截图")

工作流程：

* ProactorInitiator 创建 Proactor 和 Handler 对象，并将 Proactor 和 Handler 通过 Asynchronous Operation Processor（AsyOptProcessor）注册到内核
* AsyOptProcessor 处理注册请求，并处理 I/O 操作，完成I/O后通知 Proactor
* Proactor 根据不同的事件类型回调不同的 Handler 进行业务处理，最后由 Handler 完成业务处理

对比：Reactor 在事件发生时就通知事先注册的处理器（读写在应用程序线程中处理完成）；Proactor 是在事件发生时基于异步 I/O 完成读写操作（内核完成），I/O 完成后才回调应用程序的处理器进行业务处理

模式优点：异步 I/O 更加充分发挥 DMA（Direct Memory Access 直接内存存取）的优势

模式缺点：

* 编程复杂性，由于异步操作流程的事件的初始化和事件完成在时间和空间上都是相互分离的，因此开发异步应用程序更加复杂，应用程序还可能因为反向的流控而变得更加难以调试
* 内存使用，缓冲区在读或写操作的时间段内必须保持住，可能造成持续的不确定性，并且每个并发操作都要求有独立的缓存，Reactor 模式在 socket 准备好读或写之前是不要求开辟缓存的
* 操作系统支持，Windows 下通过 IOCP 实现了真正的异步 I/O，而在 Linux 系统下，Linux2.6 才引入异步 I/O，目前还不完善，所以在 Linux 下实现高并发网络编程都是以 Reactor 模型为主


****



### Netty

Netty 主要基于主从 Reactors 多线程模型做了一定的改进，Netty 的工作架构图：

![输入图片说明](https://foruda.gitee.com/images/1679451371851784196/cfa6e11e_8616658.png "屏幕截图")

工作流程：

1. Netty 抽象出两组线程池 BossGroup 专门负责接收客户端的连接，WorkerGroup 专门负责网络的读写

2. BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup，该 Group 相当于一个事件循环组，含有多个事件循环，每一个事件循环是 NioEventLoop，所以可以有多个线程

3. NioEventLoop 表示一个**循环处理任务的线程**，每个 NioEventLoop 都有一个 Selector，用于监听绑定在其上的 Socket 的通讯

4. 每个 Boss NioEventLoop 循环执行的步骤：

   - 轮询 accept 事件
   - 处理 accept 事件，与 client 建立连接，生成 NioScocketChannel，并将其**注册到某个 Worker 中**的某个 NioEventLoop 上的 Selector，连接就与 NioEventLoop 绑定
   - 处理任务队列的任务，即 runAllTasks

5. 每个 Worker NioEventLoop 循环执行的步骤：

   - 轮询 read、write 事件
   - 处理 I/O 事件，即 read，write 事件，在对应 NioSocketChannel 处理
   - 处理任务队列的任务，即 runAllTasks

6. 每个 Worker NioEventLoop 处理业务时，会使用 Pipeline（管道），Pipeline 中包含了 Channel，即通过 Pipeline 可以获取到对应通道，管道中维护了很多的处理器 Handler

   ![输入图片说明](https://foruda.gitee.com/images/1679451333642310455/bfd68f6b_8616658.png "屏幕截图")




***

## 组件介绍

### EventLoop

#### 基本介绍

事件循环对象 EventLoop，**本质是一个单线程执行器同时维护了一个 Selector**，有 run 方法处理 Channel 上源源不断的 IO 事件

事件循环组 EventLoopGroup 是一组 EventLoop，Channel 会调用 Boss EventLoopGroup 的 register 方法来绑定其中一个 Worker 的 EventLoop，后续这个 Channel 上的 IO 事件都由此 EventLoop 来处理，保证了事件处理时的线程安全

EventLoopGroup 类 API：

* `EventLoop next()`：获取集合中下一个 EventLoop，EventLoopGroup 实现了 Iterable 接口提供遍历 EventLoop 的能力
* `Future<?> shutdownGracefully()`：优雅关闭的方法，会首先切换 EventLoopGroup 到关闭状态从而拒绝新的任务的加入，然后在任务队列的任务都处理完成后，停止线程的运行，从而确保整体应用是在正常有序的状态下退出的

* `<T> Future<T> submit(Callable<T> task)`：提交任务
* `ScheduledFuture<?> scheduleWithFixedDelay`：提交定时任务