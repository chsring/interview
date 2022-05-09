## Linux内核部分基础
### 进程隔离/虚拟地址空间
- 进程之间是不允许互相访问的

### 系统调用
- 用户可以从应用层通过系统调用来访问内核的方法

### binder驱动
- 相当于硬件接口

## Binder通信机制介绍
- Binder是跨进程的通信机制
- 对Server来说，Binder指的是Binder本地对象，对Client来说，Binder指的是Binder代理对象
- 对传输过程，Binder是可以进行跨进程传递的对象，会自动完成服务器和客户端对象转换

### 为什么使用binder
- Android使用Linux内核，它有很多快进程通信机制，IPC通信，如管道，共享内存，信号量，socket，文件等
- 性能：广泛IPC会提出性能要求
- 安全：传统IPC没有对通信双方进行身份验证，Binder双方支持通信双方身份校验

### Binder通信模型
- 客户端持有一个服务代理，代理对象协助驱动，完成跨进程通信。
![img.png](../resource/Binder机制.png)
- 1.Server在ServiceManager，注册某方法add
- 2.Client从ServiceManager中查询是否有该方法，如果有，SM会返回Client一个代理空方法。
- 3.当Client调用该方法时，他会返回给内核驱动，内核驱动调用Server的add方法，把结果返回给驱动，驱动返回给Client

## AIDL
- Binder实例