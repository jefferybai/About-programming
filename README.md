# About-programming
## 知识点

- [编译与解释有什么区别](#编译与解释有什么区别)
- [js异步机制](#js异步机制)
- [http协议](#http协议)
- [http中get与post的区别](#http中get与post的区别)

## 编译与解释有什么区别 

传统意义上的所谓编译与解释，区别在于代码是在什么时候被翻译成目标CPU的指令，
编译型语言在编译过程中生成目标平台的指令，
解释型语言在运行过程中才生成目标平台的指令。

#### c语言（每个平台编译一次）：

编译 =》 arm平台

编译 =》 x86平台

#### java（一次编译到处运行）：

编译 =》 中间代码 =》 虚拟机解释 =》 arm平台/x86平台

#### javascript(边编译边运行，v8引擎则会提前编译解释)

编译解释 =》 arm平台/x86平台

虚拟机的任务是在运行过程中将中间代码翻译成目标平台的指令。

## js异步机制 

### 基本概念

#### 线程和进程
进程和线程都是系统资源分配和调度的单元
一个进程可以调起多个线程， 同一进程下的线程可以共享资源，一个进程资源不够用的时候可以开启多线程。

#### 并行与并发
多个cpu同时执行不同的任务就是并行。 
单核cpu切换执行不同任务就是并发。
所以多线程在多核cpu下才能发挥最大作用。

#### js为什么是单线程

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

#### 单线程的js如何处理异步

目前最为流行的浏览器为：Chrome，IE，Safari，FireFox，Opera。浏览器的内核是多线程的。一个浏览器通常由以下几个常驻的线程：

渲染引擎线程：顾名思义，该线程负责页面的渲染

JS引擎线程：负责JS的解析和执行

定时触发器线程：处理定时事件，比如setTimeout, setInterval

事件触发线程：处理DOM事件

异步http请求线程：处理http请求
需要注意的是，渲染线程和JS引擎线程是不能同时进行的。
渲染线程在执行任务的时候，JS引擎线程会被挂起。因为JS可以操作DOM，若在渲染中JS处理了DOM，浏览器可能就不知所措了。


#### js引擎同步执行
```js
var x = 10;

function foo(){
    var y=20;
    function bar(){
        var z=15; 
    }
    bar();
}

foo();
```
代码运行，首先进入全局上下文。然后执行foo()的时候，就进入了foo上下文，当然此时全局上下文还在。当执行 bar()的时候，又进入了bar上下文。执行完毕bar()，回到foo上下文。执行完foo()，又回到全局上下文。所以，执行过程执行上下文会形成一个调用栈(Call stack)，先进后出。
```js
// 进栈
                                                          3 bar Context
    =>                      =>   2 foo Context     =>     2 foo Context
        1 global Context         1 global Context         1 global Context

// 出栈
3 bar Context
2 foo Context     =>   2 foo Context     =>                        =>
1 global Context       1 global Context         1 global Context
```

#### 启动定时触发器线程
```js
var x = 10;

function foo(){
    var y=20;
    function bar(){
        var z=15; 
    }
    bar();
}

foo();
setTimeout(function runAsync() {})

function close() { console.log('close')};
close();
```
依照js同步执行的原理，foo上下文环境出栈，运行至setTimeout交给浏览器的定时处理线程, 运行到close，至close出栈。任务队列执行完毕。

当定时线程执行完setTimeout,运行runAsync(),发现执行栈为空，js引擎会将事件队列的队首出队至JS执行栈中。

以上就是JS引擎的事件循环机制，是实现异步的一种机制。主要涉及到浏览器线程，任务队列以及JS引擎。

所以，我们可以看出，JS的异步请求，借助了而它所在的运行环境浏览器来处理并且返回结果。

而且，这也解释了为什么那些回调函数的this指向window，因为这些异步的代码都是在全局上下文环境下执行的。

## http协议

#### 浏览器背后的故事

当我们在浏览器地址栏上输入要访问的URL后，浏览器会分析出URL上面的域名，然后通过DNS服务器查询出域名映射的IP地址，浏览器根据查询到的IP地址与Web服务器进行通信，而通信的协议就是HTTP协议。

我们可以把这个过程类比成一个电话对话的过程。

两个人能够正常的进行通话的必要条件是： 1，至对方电话号码； 2， 两个人有共同的语言。

在本例中，电话号码相当于上面的IP地址，而共同语言相当于HTTP协议。

#### TCP/IP协议
HTTP协议是构建在TCP/IP协议之上的，是TCP/IP协议的一个子集，所以要理解HTTP协议，有必要先了解下TCP/IP协议相关的知识。
TCP/IP协议族是由一个四层协议组成的系统，这四层分别为：应用层、传输层、网络层和数据链路层。
接下来，我们将会介绍各个层的主要作用。

##### 1) 应用层

应用层一般是我们编写的应用程序，其决定了向用户提供的应用服务。应用层可以通过系统调用与传输层进行通信。

处于应用层的协议非常多，比如：FTP（File Transfer Protocol，文件传输协议）、DNS（Domain Name System，域名系统）和我们本章讨论的HTTP（HyperText Transfer Protocol，超文本传输协议）等。

##### 2) 传输层

传输层通过系统调用向应用层提供处于网络连接中的两台计算机之间的数据传输功能。

在传输层有两个性质不同的协议：TCP（Transmission Control Protocol，传输控制协议）和UDP（User Data Protocol，用户数据报协议）。

##### 3) 网络层

网络层用来处理在网络上流动的数据包，数据包是网络传输的最小数据单位。该层规定了通过怎样的路径（传输路线）到达对方计算机，并把数据包传输给对方。

##### 4) 链路层

链路层用来处理连接网络的硬件部分，包括控制操作系统、硬件设备驱动、NIC（Network Interface Card，网络适配器）以及光纤等物理可见部分。硬件上的范畴均在链路层的作用范围之内。


而各个层级之间的通讯是在上层数据上包装一层自己的类型，按照层级依次发送。

#### TCP三次握手

第一次握手：客户端发送带有SYN标志的连接请求报文段，然后进入SYN_SEND状态，等待服务端的确认。

第二次握手：服务端接收到客户端的SYN报文段后，需要发送ACK信息对这个SYN报文段进行确认。同时，还要发送自己的SYN请求信息。服务端会将上述的信息放到一个报文段（SYN+ACK报文段）中，一并发送给客户端，此时服务端将会进入SYN_RECV状态。

第三次握手：客户端接收到服务端的SYN+ACK报文段后，会想服务端发送ACK确认报文段，这个报文段发送完毕后，客户端和服务端都进入ESTABLISHED状态，完成TCP三次握手。


当三次握手完成后，TCP协议会为连接双方维持连接状态。为了保证数据传输成功，接收端在接收到数据包后必须发送ACK报文作为确认。如果在指定的时间内（这个时间称为重新发送超时时间），发送端没有接收到接收端的ACK报文，那么就会重发超时的数据。


#### TCP四次挥手

第一次挥手： 客户端-发送一个FIN，用来关闭客户端到服务器的数据传送

第二次挥手： 服务器-收到这个FIN，它发回一个ACK，确认序号为收到的序号加1 。和SYN一样，一个FIN将占用一个序号

第三次挥手： 服务器-关闭与客户端的连接，发送一个FIN给客户端

第四次挥手： 客户端-发回ACK报文确认，并将确认序号设置为收到序号加1

参考文章： http://mp.weixin.qq.com/s/27zpNIGhVbx-on9FDs_6dw

## http中get与post的区别

在我大万维网世界中，TCP就像汽车，我们用TCP来运输数据，它很可靠，从来不会发生丢件少件的现象。但是如果路上跑的全是看起来一模一样的汽车，那这个世界看起来是一团混乱，送急件的汽车可能被前面满载货物的汽车拦堵在路上，整个交通系统一定会瘫痪。为了避免这种情况发生，交通规则HTTP诞生了。

get/post本质是tcp链接，因为http规则，和浏览器的不同也会产生一些不同的差异。

HTTP给汽车运输设定了好几个服务类别，有GET, POST, PUT, DELETE等等

（大多数）浏览器通常都会限制url长度在2K个字节，而（大多数）服务器最多处理64K大小的url。超过的部分，恕不处理。如果你用GET服务，在request body偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然GET可以带request body，也不能保证一定能被接收到哦





