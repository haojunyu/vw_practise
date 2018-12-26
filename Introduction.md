# 介绍

## 入门
C语言没有提供内置的工具来表现这些常用的操作，像输入/输出，内存管理，字符串操作等。而是在标准库中定义了这些工具以便我们对自己的程序进行编译链接。

  本文描述的GNU C库不仅定义了ISO C标准指定的所有库函数，还包含了可移植操作系统接口（POSIX）和衍生的Unix操作系统里的额外功能。
  这本手册的目的是告诉你如何使用GNU C库里的工具。并通过描述不同标准的特色功能来帮你判别那些不能移植到其他系统的东西。但是这个手册的重点不在于严格的可移植性。

## 标准和可移植性
这一部分讨论各种各样的标准以及GNU C库基于的源。包含ISO C、POSIX标准、系统V以及伯克利Unix。

  这个手册的目的是要告诉你如何充分利用GNU C库的工具。但是如果你关心的是如何使你的程序在这些标准下更加协调或者比其他GNU系统更加便捷，这个手册也能对你如何使用库产生影响。这部分将对这些标准做一个总览，所以你将知道手册其他部分章节干了啥。

  附录B提供了该库按字母排列的函数和符号。该列表也描述了每个函数和符号来自哪个标准。


### ISO标准C
GNU C库是满足C标准的并且被美国国家标准机构（ANSI）：American National Standard X3.159-1989-“ANSI C”以及国际标准组织（ISO）：ISO/IEC 9899:1990-"Programming languages-C"所接受。这里我们把这个标准称为ISO C，因为这是一个被更多人认可的标准。组成GNU C库的头文件和库工具是按照ISO C标准实例化的一个扩展集。
 
  如果你想要严格准守ISO C标准，你可以用GUN C编译器编译你的程序时使用`-ansi`参数。该参数告诉编译器从头文件中使用ISO 标准来编译，除非你特地请求使用其他的特色。章节1.3.4会告诉你如何做。
 
  这个手册无法给出ISO C和其他版本的差异细节，但是它对于你在多个C版本的情况下写出可移植性的程序是很有帮助的。

### POSIX（可移植操作系统接口）
GNU C库也是兼容ISO POSIX的标准-计算机环境的可移植操作系统接口（ISO/IEC9945）.这个标准也叫ANSI/IEEE Std 1003。POSIX源自于各种Unix操作系统版本。

  POSIX标准规定的库功能是ISO C标准的扩展集；POSIX为ISO C函数规定了额外的特色，也增加了新的函数。一般情况下，POSIX标准增加的需求和函数定义都是为特定的操作系统环境提供低水平的支持，而不是为不同操作系统环境里面提供一般的编程语言支持。

  GNU C库实现了ISO/IEC 9945-1:1996（[System Application Program Interface](http://ieeexplore.ieee.org/document/846694/?reload=true)，也叫POSIX.1）定义的所有函数。它对于ISO C标准的扩展主要在以下方面，包括文件系统接口（详情见第14章-文件系统接口），具体设备终端控制函数（详情见17章-低水平终端接口）以及进程控制函数（详情见26章-进程）。

  ISO/IEC 9945-2:1993中的一些功能，POSIX外壳和标准实用工具（POSIX.2）也被GNU C实现了。这些使用工具包含处理正则表达式和其他模式匹配工具（详情见第十章-模式匹配）。

#### POSIX安全理论
本手册对于GNU C库函数的安全等级进行了初步的区分，下面是它们的安全等级：多线程安全（MT-Safe），异步信号安全（AS-Safe），异步取消安全（AC-Safe）。

  这个安全等级的初步划分是按照POSIX提出的原则提出的。下面是这三个等级的直观定义（不是标准定义）：
* 多线程安全
多线程安全函数在其他线程中的调用是安全的。要做到多线程安全并不是意味着这个函数是原子的，也并不意味着它使用了POSIX提供给用户的内存同步机制。顺序的调用多线程安全函数而不是产生一个多线程安全组合也是一种可能。比如说，一个线程依次无缝调用两个多线程安全函数。这种情况和两个函数依次组成一个原子操作是不同的，因为后者会被其他线程的同时调用严重影响。
> **TIPS:**
> 假如两个线程都是按顺序执行函数A和B，不过线程1中函数A和B都是多线程安全的，而线程2中函数A和B都不是多线程安全的，但是A和B的组合是原子性的。这两种情况，线程1是多线程安全的，而线程2不是多线程安全的。

  Whole-program optimizations that could inline functions across library interfaces may expose unsafe reordering, and so performing inlining across the GNU C Library inter- face is not recommended. The documented MT-Safety status is not guaranteed under whole-program optimization. However, functions defined in user-visible headers are designed to be safe for inlining.
> **TIPS:**
> 这段讲的是什么样的情形

* 异步信号安全
异步信号安全函数在异步信号处理程序的调用中是安全的。许多异步信号安全函数可能设置了`errno`,也可能修改了浮点环境，因为这种做法可以使得该函数能够适用在信号处理程序中。然而如果异步信号处理程序修改了本地线程的状态，那么程序就会失常，而且这种情况是信号处理机制无法预防的。因此，信号处理程序会在调用函数前通过设置`errno`或者修改浮点环境来保存本地线程原始状态并且在函数返回前恢复之前的状态。
> **TIPS:**
> 为什么设置`errno`或修改浮点环境可以使函数适用信号处理程序
  
* 异步取消安全
异步取消安全函数在异步取消被启用时调用是安全的。POSIX标准只定义了三个函数是异步取消安全的，分别是`pthread_cancel`，`pthread_setcancelstate`，`pthread_setcanceltype`。当前，GNU C库无法保证只有这三个函数，但确实记录下了当前哪些韩式是异步取消安全的。这个文档仅供GNU C库开发者使用。

  就像信号处理程序，取消清除程序必须配置它们所需的浮点环境。 程序不能假定当前的环境是浮点环境，特别是当异步取消启动的情况下。 而且如果浮点环境的配置无法以原子方式执行，那么所碰到环境也是和预想的不一致。

* 多线程不安全，异步信号不安全，异步取消不安全
这三种不安全函数在上面提到的安全环境中调用是不安全的。在安全环境中调用它们会引发不确定的事情。函数在安全环境中没有被明确的定义为安全就应该被当做不安全的。

* 初步安全属性
初步安全属性被记录在案，意味着这些属性在未来的GNU C库发行版本可能不会被排除。这些属性是对我们当前实现函数的评估结果而不是当前和未来的标准规定和允许的。
> **TIPS:**
> 我们都想所有的函数都是多线程安全的，异步信号安全的，异步取消安全的，但可能受限于当前的框架，当前的理念，当前的方法，有的函数只能实现成满足其中之一，之二或一个都不满足。而且我们对于哪些函数应该是哪种安全目前或未来的标准都没有明确标准，我们只能对着已经实现的函数说它是[多线程|异步信号|异步取消|不]安全的。

  虽然我们努力遵守标准，但是在某些情况下，即使标准不要求安全，我们的实现也是安全的。而在其他情况下，我们的实现不符合标准的安全要求。后者很可能是个Bug;而被标记的初步安全属性不应该约束未来的标准，即未来的标准和以当前实现为支撑的安全属性有冲突时以未来的标准为准。

  此外，POSIX标准并没有提供对安全作一个详细的定义。我们假定“安全的调用”是指程序不会引发未定义的行为，安全调用的函数能够实现我们指定的任务并且不会导致其他函数偏离其指定的行为。这里我们选择使用宽松的安全定义不是因为它们是最好的定义，而是因为它们和POSIX标准更加和谐一致。

  请注意，这些都初步的定义和解释，并且定义的某些方面还在讨论中而且以后还会进行澄清或更正。

  随着时间的转移，我们设想将这初步的安全属性变成像函数接口一样稳定的属性。一旦我们做到了，我们将会从安全属性上去掉`Preliminary`关键字。不过只要这个关键字还在，它就会被当做对未来行为的承诺。

  安全方面其他关键字的定义将会在下面的章节中出现。


#### 不安全功能
在特定环境中不能安全调用的函数已经被关键字标注了。这这部分，异步信号不安全的函数意味着这个函数在异步信号被激活时是不安全的。而异步取消不安全的函数意味着这个函数在异步取消被激活时也是不安全的。不过这部分没有多线程不安全的标注。
* `lock`
在异步信号不安全的关键字中，`lock`关键字意味着被它标注并且拥有非递归锁的函数会被信号打断。如果信号处理程序同时调用另一个拥有相同锁的函数，此时就会发生死锁。

Functions annotated with lock as an AC-Unsafe feature may, if cancelled asynchronously, fail to release a lock that would have been released if their execution had not been interrupted by asynchronous thread cancellation. Once a lock is left taken, attempts to take that lock will block indefinitely.
> **TIPS:**
> lock对AC-Unsafe情况的场景。
>


* `corrupt`
Functions marked with corrupt as an AS-Unsafe feature may corrupt data structures and misbehave when they interrupt, or are interrupted by, another such function. Unlike functions marked with lock, these take recursive locks to avoid MT-Safety problems, but this is not enough to stop a signal handler from observing a partially- updated data structure. Further corruption may arise from the interrupted function’s failure to notice updates made by signal handlers.

  Functions marked with corrupt as an AC-Unsafe feature may leave data structures in a corrupt, partially updated state. Subsequent uses of the data structure may misbehave.

* `heap`
被关键字`heap`标注的函数可能会从`malloc/free`家族函数中调用堆内存管理函数，他们和这些家族函数一样安全。这个标注等价于|`AS-Unsafe lock`|`AC-Unsafe lock fd mem`|

* `dlopen`
被关键字`dlopen`标记的函数会使用动态加载器来将共享库
加载进当前执行环境{current execution image}中。它包含打开文件，映射内容到内存中，申请额外空间，解析符号，应用重定位等，所有这些都会在内部动态加载锁的状态下执行。

  这些锁定的时间足够使函数成为异步信号或异步取消不安全，而且可能会出现其他问题。当前，关键字`dlopen`就是针对这些由`dlopen`引起的潜在安全问题的概述。

* `plugin`
被关键字`plugin`标记的函数可能运行来自外部插件的代码，外部是相对于GNU C库而言。假定这些插件函数是多线程安全，异步信号不安全和异步取消不安全的，比如说栈展开库`stack unwinding library`，`name service switch`和字符集转换`iconv`等插件.

  虽然上面例子中的插件都是通过`dlopen`引入的，但是关键字`plugin`并不意味着和动态加载或`libdl`接口相关，它是被`dlopen`包含的。换句话说，如果一个函数装载一个模块，并找到了其中一些函数的地址，而此时另一个函数就调用这些已经解析好的函数，前者会被`dlopen`标记，而后者将会被`plugin`标记。如果一个函数既要装载一个模块，也要调用装载模块中的函数，那么它将被这两个关键字标记。

* `i18n`
被关键字`i18n`标记的函数可能会调用gettext库中的国际化函数并且会和这些函数一样安全。所以这部分的注解等价于|`MT-Safe env` | AS-Unsafe corrupt heap dlopen` | `AC-Unsafe corrupt`|

* timer
被关键字`timer`标记的函数会使用报警`alarm`函数或者为系统调用以及耗时的操作设置定时器（tme-out）。在多线程的程序中，这是一个风险，因为超时信号会被送给处理超时信号的线程，因此无法中断之前预期的线程。该函数除了是多线程不安全之外，它也总是异步信号不安全的，因为在信号处理程序中调用它们会干扰中断代码中设定的定时器；此外它们也总是异步取消不安全的，因为没有安全的方法来保证异步取消的情况下较早的计时器会被重置。

#### 有条件的安全功能
#### 其他安全备注
### 伯克利Unix
伯克利Unix是伯克利软件套件（Berkeley Software Distribution，BSD），是一个操作系统的名称。
### SVID（系统V接口描述）
### XPG（开放便携性指南）
## 库的使用
### 头文件
### 函数的宏定义
### 保留的名称
### 功能测试宏
## 章节说明


