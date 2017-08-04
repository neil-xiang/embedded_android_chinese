> 翻译：[Neil-xiang](https://github.com/Neil-xiang)
> 校对：

###AOSP包(Stock AOSP Packages)
	AOSP中包含有大量在大部分的Android设备中能找到的默认包。如我在之前的章节提到的那样，虽然有一些诸如Maps、YouTube、Gmail并不是AOSP中的一部分。让我们来看看默认包括的一些值得一提的包；像
我们下面看到的那样，SOAP包含了大量的包。表2-7列出了Android2.3/Gingerbread AOSP中的最重要的应用app；表2-8列出了AOSP的主要内容提供商；表2-9列出了对应的IME(input method editors)。
	
			**应用程序就像通用的app，但是不会在AOSP外使用标准SDK构建。所以，如果你想创建其中一个app的你自己的版本，你可以在AOSP内部修改或者花费时间使用标准SDK在AOSP中构建一个app，值得注意的
				是，这些app中应用的API只能在AOSP中才能访问，在标准的SDK中无法访问。
	AOSP包含了比上面表中列出来的多得多的包。实际上，如果你搜索源代码，你会发现Android4.2/Jelly Bean可以产生大约500个app。这些app中的大部分要么是测试使用的要么是一个事例，并不值得在现在花
费时间来讨论。大约有1/4的app值得放入最终的产品中，而且你几乎可以在AOSP以下目录中找到它们：
	• packages/apps/
	• packages/inputmethods/
	• packages/providers/
	• packages/screensavers/ (new to 4.2/Jelly Bean)
	• packages/wallpapers/
	• frameworks/base/packages/
	• development/apps/
	你可能想要结合上述表格查看这些目录的内容来决定哪些包值得在你的项目中进一步的了解。就像AOSP中的大多数其他事物一样，软件包也会随着时间改变或者改变路径。以下是2.3.4/Gingerbread和4.2/Jelly Bean
之间发生的一些位置更改的摘要：


###系统启动(System Startup)
	将我们讨论过的一切集合起来的最佳方法是看Android的启动。就像你在图2-6中看到的一样，第一个齿轮是CPU，通常从一个硬件编码的地址取出第一条指令。这个地址通常指向芯片bootloader程序的地址。bootloader
初始化RAM，将基本的硬件设置为静止状态，导入内核及RAM，并跳转到内核。进来的很多Soc设备，在一颗芯片中包含了CPU和很多的外设，事实上可以直接从格式化的SD卡或者类似SD卡的芯片启动。例如PandaBoard最近的
版本BeagleBoard,因为直接从SD卡启动，在板上就没有任何的flash芯片。
	*图2-6 Android的启动流程

	初始内核启动非常依赖于硬件，但它的的目的是设置好事情以便CPU可以尽早的开始执行C代码。一旦这个完成后，内核就跳转到不依赖指令集(architecture-independents)的start_kernel()函数，初始化各种子系统并调
用内置驱动的‘init’功能。在起始阶段内核打印的大量信息来源于这些步骤。此时内核挂载它的根文件系统并开始init进程。

	这是Android的init开始执行存放在它/init.rc文件中的指令来设置各种环境变量，比如系统路径、创建挂载点、挂载文件系统、设置OOM的策略，开始native 后台。我们已经讲了很多Android中的native后台进程活动了，
但是Zygote仍然是一个值得小小注意的地方。Zygote是一个launch app的特殊的后台进程。它的功能主要集中在统一所有app共用的组件并缩短他们的启动时间。init实际上不直接开始Zygote；而是由Android Runtime使用
app_process命令来启动Zygote。runtime这是开始系统的第一个Dalvik VM并使用它调用Zygote的main().

	Zygote只在一个新的app需要launched的时候才处于活动状态。为了达到更快的app启动速度，Zygote首先预加载应用程序在运行时可能需要的所有Java类和资源。高效的将这些资源放入系统RAM中。然后Zygote会在它的端
口(dev/socket/zygote)上监听开始新app的请求。当得到一个开始新的app的请求时，它将自己复制并启动新的app，所有的app从Zygote分叉的美妙之处是它是一个全新的虚拟机，所有app可能使用到的系统类和资源都已经被
预加载并随时可以被使用。换句话所，新的app不需要得到这些被加载才开始执行。
	
	所有这些能工作是因为Linux内核为分叉提供了写时备份(copy-on-write(COW))机制。你可能知道，涉及到Unix中的分叉是实际上从父进程中创建一个新的进程。而有了COW，Linux就不需要真正的拷贝任何东西了。相反，它
将新进程的页面映射到父进程的页面，并且只有当新进程写入页面时才复制副本。但是实际上，这些加载的类及资源从来不会被改写，因为它们是默认的，并且在系统的生命周期内几乎是不可变的。所以直接从Zygote分配的所
有进程基本上都是使用自己的映射副本。因此，不管系统中运行了多少个app，系统类及资源都只有加载到RAM中的那唯一一份副本。

	虽然Zygote被设计为监听请求已开始一个新的app，但是Zygote却显然开始了一个‘app’:系统服务。这是Zygote开始的第一个app，它确完全的与他的父进程分割而持续存在。然后，系统服务器开始初始化其所有的每个系统服
务，并将其注册到先前启动的服务管理器。它开始的一项服务:Activity Manager,将发送一个Intent.CATEGORY_HOME类型的意图来结束初始化。然后将启动Launcher应用程序，然后显示所有Android用户熟悉的主屏幕。

	当用户点击主屏幕上的一个图标是，我在“A Service Example: the Activity Manager”中描述过的进程会启动。Launcher会告诉Activity Manager启动进程，Activity Manager会将这个请求转发给Zygote，Zygote会启动一个新
的app，然后显示给用户。

	一旦系统启动完成，进程列表看起来会像这样:
	# ps
	USER PID PPID VSIZE RSS WCHAN PC NAME
	root 1 0 268 180 c009b74c 0000875c S /init
	root 2 0 0 0 c004e72c 00000000 S kthreadd
	root 3 2 0 0 c003fdc8 00000000 S ksoftirqd/0
	root 4 2 0 0 c004b2c4 00000000 S events/0
	root 5 2 0 0 c004b2c4 00000000 S khelper
	root 6 2 0 0 c004b2c4 00000000 S suspend
	root 7 2 0 0 c004b2c4 00000000 S kblockd/0
	root 8 2 0 0 c004b2c4 00000000 S cqueue
	root 9 2 0 0 c018179c 00000000 S kseriod
	root 10 2 0 0 c004b2c4 00000000 S kmmcd
	root 11 2 0 0 c006fc74 00000000 S pdflush
	root 12 2 0 0 c006fc74 00000000 S pdflush
	root 13 2 0 0 c0079750 00000000 D kswapd0
	root 14 2 0 0 c004b2c4 00000000 S aio/0
	root 22 2 0 0 c017ef48 00000000 S mtdblockd
	root 23 2 0 0 c004b2c4 00000000 S kstriped
	root 24 2 0 0 c004b2c4 00000000 S hid_compat
	root 25 2 0 0 c004b2c4 00000000 S rpciod/0
	root 26 1 232 136 c009b74c 0000875c S /sbin/ueventd
	system 27 1 804 216 c01a94a4 afd0b6fc S /system/bin/servicemanager
	root 28 1 3864 308 ffffffff afd0bdac S /system/bin/vold
	root 29 1 3836 304 ffffffff afd0bdac S /system/bin/netd
	root 30 1 664 192 c01b52b4 afd0c0cc S /system/bin/debuggerd
	radio 31 1 5396 440 ffffffff afd0bdac S /system/bin/rild
	root 32 1 60832 16348 c009b74c afd0b844 S zygote
	media 33 1 17976 1104 ffffffff afd0b6fc S /system/bin/mediaserver
	bluetooth 34 1 1256 280 c009b74c afd0c59c S /system/bin/dbus-daemon
	root 35 1 812 232 c02181f4 afd0b45c S /system/bin/installd
	keystore 36 1 1744 212 c01b52b4 afd0c0cc S /system/bin/keystore
	root 38 1 824 272 c00b8fec afd0c51c S /system/bin/qemud
	shell 40 1 732 204 c0158eb0 afd0b45c S /system/bin/sh
	root 41 1 3368 172 ffffffff 00008294 S /sbin/adbd
	system 65 32 123128 25232 ffffffff afd0b6fc S system_server
	app_15 115 32 77232 17576 ffffffff afd0c51c S com.android.inputmethod.
	latin
	radio 120 32 86060 17952 ffffffff afd0c51c S com.android.phone
	system 122 32 73160 17656 ffffffff afd0c51c S com.android.systemui
	app_27 125 32 80664 22900 ffffffff afd0c51c S com.android.launcher
	app_5 173 32 74404 18024 ffffffff afd0c51c S android.process.acore
	app_2 212 32 73112 17032 ffffffff afd0c51c S android.process.media
	app_19 284 32 70336 16672 ffffffff afd0c51c S com.android.bluetooth
	app_22 292 32 72752 17844 ffffffff afd0c51c S com.android.email
	app_23 320 32 70276 15792 ffffffff afd0c51c S com.android.music
	app_28 328 32 70744 16444 ffffffff afd0c51c S com.android.quicksearchbox
	app_14 345 32 69708 15404 ffffffff afd0c51c S com.android.protips
	app_21 354 32 70912 17152 ffffffff afd0c51c S com.cooliris.media
	root 366 41 2128 292 c003da38 00110c84 S /bin/sh
	root 367 366 888 324 00000000 afd0b45c R /system/bin/ps
	
	这实际上来源与Android2.3/Gingerbread的Android模拟器，因此，它包含一些模拟器特征的产物，比如‘qemud’后台进程。请注意，即使从Zygote分化，所有运行的应用程序都承担了完全合格的包名称。通过使用prctl()系统
调用和R_SET_NAME的来告诉内核更改调用进程的名称，这是一个从Linux拿来的巧妙的花招。如果你对这个有兴趣，可以查看prctl()的参考页。还请注意，‘Init’启动的第一个进程实际上是‘ueventd’。所有在它之前的进程实际
上都是在内核由子系统或驱动启动的。
	
	最重要的是注意Zygote的进程ID(PID)是32，所以所有app的父进程的PID(PPID)也是32.这证明了早期关于Zygote是系统中所有app的父进程的说明。