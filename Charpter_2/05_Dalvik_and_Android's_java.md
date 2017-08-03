> 翻译：[Neil-xiang](https://github.com/Neil-xiang)
> 校对：

###Dalvik虚拟机和Android的Java(Dalvik and Android’s Java)
	简而言之，Dalvik是Android的Java虚拟机。它允许Android运行基于java的字节代码应用及Android自己的系统组件，并提供所需的钩子和环境，以与系统的其余部分进行交互。包括本机库和本机用户空间
的其余部分。不过，还有更多关于Dalvik和Android的Java。但是在我深入解释这个之前，我首先要介绍一下Java的基础知识。
	为了避免关于Java语言及其起源的历史课会让你感到无聊，一句话来说就是Java是James Gosling在90年代早期在Sun时创造出来的，它迅速变得非常受欢迎，而且总之，在Android来临之前，它已经非常成
熟。从开发人员的角度来看，关于Java需要牢记两个方面:它和c、C++等传统的语言不同；组成了我们常称为‘java’的组件。
	和c及c++代码被编译器编译为汇编二进制文件并在符合编译器指令集的CPU上运行不同，Java被设计成解释型语言。你在Java中编写的代码由Java编译器编译成与指令集无关的字节代码，并在运行时由字节
代码解释器(通常称为’虚拟机(virtual machine)‘)执行.这种操作方式以及Java的语义使得该语言能够包含以前传统语言中没有的很多功能，比如映像(reflction)及匿名类(anonymous classes)。此外，与C
和C++不同，Java不要求你跟踪你分配的对象。实际上，它要求你丢掉所有对未使用对象的跟踪，因为它有一个集成的垃圾收集器，将确保所有这些对象在没有活动代码引用他们是被破坏。
	实际上，Java由几个不同的东西组成:java编译器、java解释器(通常称为’java虚拟机(Java Virtual Machine(JVM))‘)以及Java开发人员通常使用的Java库。通常开发人员通过Oracle免费提供的Java开发工
具包(Java Development Kit(JDK))获得。Android在构建时实际上依赖于JDK作为Java编译器，但是它并没有使用JVM或JDK中发现的库。它依赖于Dalvik而不是JVM，依赖于Apache Harmony项目而不是JDK库文件，
java库中的clean-room行为在Apache项目保护下实施（？）。
		**在由AOSP构建生成的镜像中找不到JDK组件。因此，当你的嵌入式系统使用Android时，你不能发布任何的JDK组件。
		**java lingo
		Java有自己的专门术语。如果你还不熟悉他们的话，以下说明可以帮助您理解文本中使用的某些术语：
		***虚拟机(virtual machine)
		当Java出来时，这个术语有一点模糊，因为VMware和VirtualBox等“虚拟机”软件产品并不像现在那样普遍或受欢迎。像Java虚拟机那样，这样的虚拟机不仅仅是解释字节码。
		***(映射)reflection
		询问对象是否有实现某种方法的能力。
		***匿名类(anonymous classes)
		作为参数传递给正在调用的方法的代码片段。例如，可以使用匿名类作为回调注册方法，从而使开发人员可以在源代码中调用回调注册方法的同一位置查看处理事件的代码。
		***.jar文件(.jar files)
		.jar文件实际上是包含许多.class文件的java存档(Java ARchives(JAR))。每个都只包含一个类。
		
	据其开发人员Dan Bornstein介绍，Dalvik通过专门为嵌入式系统设计而与JVM区分开来。也就是说，它针对的是具有缓慢的CPU和相对较少的RAM的系统，运行不使用交换空间的
操作系统，并且是电池供电的。
	当JVM使用.class文件时，Dalvik更喜欢.dex。.dex文件实际上是通过Android的'dx'功能使用java编译器编译的.class文件再编译后产生的。除了其他方面，未压缩的.dex文件
比起始的.jar文件小50％。
	有关Dalvik的功能和内部结构的更多信息，我强烈建议您查看Dan Bornstein的Google I/O 2008演示文稿，题目是“Dalvik Virtual Machine Internals”。大概有一个小时的时
间，可以在YouTube上播放。你也可以去YouTube去搜索“Dan Bornstein Dalvik”。
		**另一个有趣的事实是，Dalvik是基于注册的，而JVM是基于堆栈的，尽管这可能对您没有任何意义，除非您是VM理论，架构和内部人员的狂热学生。
		如果您想了解基于堆栈的虚拟机和基于注册的虚拟机之间的利益和权衡，请参阅Shi等人的题为“虚拟机展示：堆栈与注册表”的论文 2005年6月11日至12日，VEE'05的诉讼，芝加哥，153-163
	Dalvik的一个特点非常值得强调的是，自从Android2.2/Froyo以来，它已经为ARM提供了一个Just-in-Time（JIT）编译器，其中x86和MIPS也已经有添加。历史上，JIT一直是许
多虚拟机的一个定义功能，帮助他们弥补与非解释语言的差距。事实上，拥有JIT意味着Dalvik将应用程序的字节码转换为在目标的CPU上运行的二进制汇编指令，而不是一次由VM
解释一条指令。然后将此转换的结果存储以供将来使用。因此，应用程序第一次加载时间较长，但一旦JIT已经加载，它们的加载和运行速度会快很多。这里唯一需要注意的是，
JIT仅适用于有限数量的架构，即ARM，x86和MIPS。
	作为嵌入式开发人员，你不太可能需要做任何具体操作就能使Dalvik在您的系统上工作。Dalvik被设计为指令无关的。据报道，一些早期的Dalvik遭遇了一些大小端的问题。但是，
这些问题似乎已经解决了。

	##Java本机接口(Java Native Interface (JNI))
	尽管Java的功能和优势，Java并不总是在运行在密闭环境中,而用Java编写的代码有时需要与来自其他语言的代码进行交互。这在诸如Android的嵌入式环境中尤其如此，其中低级功能永远不会太遥远。为此，
提供了Java本机接口（JNI）机制。它本质上是一个其他语言（如C和C ++）的桥梁。它相当于.NET / C＃世界中的P / Invoke。
	应用程序开发人员有时使用JNI从使用SDK构建的常规Java代码中调用NDK编译的本机代码。然而，在内部，AOSP大量依赖JNI来使Java编码的服务和组件能够与Android的底层功能相关联，它通常是使用c/c++
来编写的。例如，Java编写的系统服务通常使用JNI与匹配的本地代码进行通信，该本地代码与给定服务相对应硬件进行交互。
	通过JNI可以让Java与其他语言沟通的重要举措实际上是由Dalvik完成的。例如，如果您回到上一节中的表2-3，您会注意到libnativehelper.so库，它作为Dalvik的一部分提供，用于促进JNI调用。
	附录B显示了JNI接口Java和C代码的示例使用。目前，请记住，JNI是Android平台工作的核心，它可能是一个相对复杂的机制，特别是要确保你使用适当的调用语义和函数参数。
		**不幸的是，JNI是从开始就是一个保留的黑暗艺术。换句话说，找到好的文档是很困难的。有一本关于这个话题的权威书，<Java本地接口程序员指南和规范>

	