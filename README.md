# About-programming
## 知识点

- [编译与解释有什么区别](#编译与解释有什么区别)
- [js异步机制](#js异步机制)

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
多线程就是并行。 
单核cpu切换执行不同任务就是并发。

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
