# About-programming
## 知识点

- [编译与解释有什么区别](#编译与解释有什么区别)
- [js中的堆内存栈内存](#js中的堆内存栈内存)
- [js内存开辟和释放](#js内存开辟和释放)
- [js异步机制](#js异步机制)
- [js微任务宏任务](#js微任务宏任务)
- [http协议](#http协议)
- [http headers](#headers)
- [http Cache-Control](#Cache-Control)
- [meta标签总结](#meta标签总结)
- [http中get与post的区别](#http中get与post的区别)
- [html页面的渲染流程](#页面渲染流程)
- [html页面重回和回流](#页面重绘和回流)
- [link标签](#link标签)
- [script标签](#script标签)


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

## js中的堆内存栈内存

基本类型就是保存在栈内存中的简单数据段，而引用类型指的是那些保存在堆内存中的对象。

### 为什么会有栈内存和堆内存之分？

   通常与垃圾回收机制有关。为了使程序运行时占用的内存最小。

   当一个方法执行时，每个方法都会建立自己的内存栈，在这个方法内定义的变量将会逐个放入这块栈内存里，栈内存的变量如果是引用类型，则存放引用内向的地址，随着方法的执行结束，这个方法的内存栈也将自然销了。因此，所有在方法中定义的变量都是放在栈内存中的；

   当我们在程序中创建一个对象时，这个对象将被保存到运行时数据区中，以便反复利用（因为对象的创建成本通常较大），这个运行时数据区就是堆内存。堆内存中的对象不会随方法的结束而销毁，即使方法结束后，这个对象还可能被另一个引用变量所引用（方法的参数传递时很常见），则这个对象依然不会被销毁，只有当一个对象没有任何引用变量引用它时，系统的垃圾回收机制才会在核实的时候回收它。
   
## js内存开辟和释放

代码执行， 进入全局作用域，开辟一块栈内存， 变量提升，代码自上而下执行， 遇到引用类型开辟一块堆内存 。 遇到方法执行会先开辟一块栈内存->形成一块私有作用域->形参赋值->变量提升->代码自上而下执行。 方法执行完毕如果没有外部引用指向方法私有作用域里面的变量那么释放栈内存私有作用域消失，反之形成闭包。

参见如下代码
```js
var a = 9;
function fn() {
   a = 0;
   return function(b) {
      return b + a ++
   }
}
var f = fn();

```

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

当定时线程执行完setTimeout,运行runAsync(),发现执行栈为空，js引擎会将宏任务事件队列的队首出队至JS执行栈中。

以上就是JS引擎的事件循环机制，是实现异步的一种机制。主要涉及到浏览器线程，任务队列以及JS引擎。

所以，我们可以看出，JS的异步请求，借助了而它所在的运行环境浏览器来处理并且返回结果。

而且，这也解释了为什么那些回调函数的this指向window，因为这些异步的代码都是在全局上下文环境下执行的。

## js微任务宏任务
macrotask 和 microtask 表示异步任务的两种分类。

在挂起任务时，JS 引擎会将所有任务按照类别分到这两个队列中，首先在 macrotask 的队列（这个队列也被叫做 task queue）中取出第一个任务，执行完毕后取出 microtask 队列中的所有任务顺序执行；之后再取 macrotask 任务，周而复始，直至两个队列的任务都取完。
![avatar](https://user-gold-cdn.xitu.io/2018/11/23/16740fa4cd9c6937?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


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

## headers

https://www.cnblogs.com/benbenfishfish/p/5821091.html

## Cache-Control

 public    ---- 数据内容皆被储存起来，就连有密码保护的网页也储存，安全性很低
 
 private    ---- 数据内容只能被储存到私有的cache，仅对某个用户有效，不能共享
 
 no-cache    ---- 可以缓存，但是只有在跟WEB服务器验证了其有效后，才能返回给客户端(如果没过期200(from cache) 如果过期了if-match对比eTag, 或者Last-Modified，如果相等304

 no-store  ---- 请求和响应都禁止被缓存
 
 max-age：   ----- 本响应包含的对象的过期时间
 Must-revalidate    ---- 如果缓存过期了，会再次和原来的服务器确定是否为最新数据，而不是和中间的proxy

 max-stale  ----  允许读取过期时间必须小于max-stale 值的缓存对象。 

 proxy-revalidate  ---- 与Must-revalidate类似，区别在于：proxy-revalidate要排除掉用户代理的缓存的。即其规则并不应用于用户代理的本地缓存上。

 s-maxage  ---- 与max-age的唯一区别是,s-maxage仅仅应用于共享缓存.而不应用于用户代理的本地缓存等针对单用户的缓存. 另外,s-maxage的优先级要高于max-age.

 no-transform   ---- 告知代理,不要更改媒体类型,比如jpg,被你改成png.
 
 ## meta标签总结
 
 https://www.cnblogs.com/heyiming/p/6229395.html


## http中get与post的区别

在我大万维网世界中，TCP就像汽车，我们用TCP来运输数据，它很可靠，从来不会发生丢件少件的现象。但是如果路上跑的全是看起来一模一样的汽车，那这个世界看起来是一团混乱，送急件的汽车可能被前面满载货物的汽车拦堵在路上，整个交通系统一定会瘫痪。为了避免这种情况发生，交通规则HTTP诞生了。

get/post本质是tcp链接，因为http规则，和浏览器的不同也会产生一些不同的差异。

HTTP给汽车运输设定了好几个服务类别，有GET, POST, PUT, DELETE等等

（大多数）浏览器通常都会限制url长度在2K个字节，而（大多数）服务器最多处理64K大小的url。超过的部分，恕不处理。如果你用GET服务，在request body偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然GET可以带request body，也不能保证一定能被接收到哦


## 页面渲染流程
解析Dom树 =》 加载Css文件 =》 解析Css树 =》 合成渲染树 =》 计算元素的布局信息 =》 布局信息和颜色信息发给GPU渲染

## 页面重绘和回流

重绘：改变外观和不改变布局，比如说字体的color值发生变化

回流：元素的布局发生变化叫做回流，回流会重绘Dom树造成较大的性能损失

回流回引起重绘，而重绘不会引起回流

回流的触发条件： 1，改变window大小 2，改变字体大小 3，添加删除元素 4，浮动或插入元素 

重绘触发条件： 1，改变字体颜色 2，改变背景色 3，visibility值的变化

减少回流的方法

使用 visibility 替换 display: none ，因为前者只会引起重绘，后者会引发回流（改变了布局）

不要使用 table 布局，可能很小的一个小改动会造成整个 table 的重新布局

将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 video或者canvas 标签来说，浏览器会自动将该节点变为图层。

## link标签
当解析htm文件时候遇到link标签，浏览器会新开一个线程去加载link标签，不会影响主线程的执行

## script标签
script标签为同步执行标签，script标签里面的代码，或者src指向的文件都是在主线程里面同步执行，所以script标签会阻塞主线程。

但是script标签有两个异步执行属性：

async: 代码异步下载，下载完成立刻执行。

defer： 代码异步下载，下载完成后等待DOMContentLoaded事件，而后依次执行script标签里面的代码。







