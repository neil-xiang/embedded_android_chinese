> 翻译：[neil-xiang](https://github.com/neil-xiang)

> 校对：

### 使用ADB(Using the Android Debug Bridge (ADB))
	Android开发团队开发环境最有趣的一个方面就是您可以使用adb工具侵入运行的仿真器或通过USB连接的任何实际设备：
	$ adb shell
	* daemon not running. starting it now on port 5037 *
	* daemon started successfully *
	# cat /proc/cpuinfo
	Processor : ARM926EJ-S rev 5 (v5l)
	BogoMIPS : 405.50
	Features : swp half thumb fastmult vfp edsp java
	CPU implementer : 0x41
	CPU architecture: 5TEJ
	CPU variant : 0x0
	CPU part : 0x926
	CPU revision : 5
	Hardware : Goldfish
	Revision : 0000
	Serial : 0000000000000000
	
	正如你所看到的，在仿真器中运行的内核报告说它正在看到一个ARM处理器，这实际上是与Android一起使用的主要平台。另外，内核说它正在一个叫做金鱼的平台上运行。这是模拟器的代码名称，你会在很多
地方看到它。
	现在你有一个shell进入模拟器，你是root，这是模拟器中的默认值，您可以运行任何命令，就像您已经在远程机器或传统的网络连接的嵌入式Linux系统中进行的操作一样。Android调试桥(ADB)使这成为可能。
要退出ADB shell会话，您需要做的只是键入Ctrl-D：

	当你在主机上首次启动adb时，它会在后台启动一个服务器，其任务是管理与连接到主机的所有Android设备的连接。这是早期输出的一部分，表示一个后台进程是在5037端口上启动的。你实际上可以要求该进
程看到哪些设备：	
	$ adb devices
	List of devices attached
	emulator-5554 device
	0000021459584822 device
	emulator-5556 offline
	这是运行一个仿真器实例的输出，一个通过USB连接的设备和另一个仿真器实例启动。如果连接了多台设备，您可以使用’-s‘标志来告知你要与哪个设备通话，以识别设备的序列号：
	$ adb -s 0000021459584822 shell
	$ id
	uid=2000(shell) gid=2000(shell) groups=1003(graphics),1004(input), ...
	$ su
	su: permission denied
	
	请注意，在这种情况下，我我的shell得到一个’$‘提示而不是一个‘＃’。这意味着与早期的交互相反，我不以root身份运行，从id命令的输出也可以看出。这实际上是一个真正的商
业Android手机，而我以前无法使用su命令获得root权限是典型的。因此，我对此设备进行任何修改的能力将相当有限。当然，除非我找到一些方法来“root”电话（即获得root权限）。
	历史上，设备制造商已经因为各种原因而非常不情愿地使用他们的设备的root权限，并且已经提出了一些规定，尽可能的困难，甚至是不可能的。这就是为什么“rooting”设备被许多
强大的用户和黑客认为是圣杯。截至2013年初，摩托罗拉，HTC和索尼移动等一些制造商已经阐明了政策变化，这些变化似乎旨在使用户更容易'root'其设备，以及注意事项。但这不是
主流。不幸的是，它受到网络运营商的挑战，他们仍然可以决定锁定手机制造商解锁的设备。

		**您可能会试图‘root’商业手机或设备进行Android平台开发实验。我建议你仔细想想。虽然有很多说明，说明如何将标准镜像替换为通常称为“自定义ROM”（如CyanogenMod等）的
		标准镜像，但您需要注意，任何错误的步骤都可能导致设备“bricking” （即，使其无法启动或擦除关键的引导时代码）。然后你有一个昂贵的纸重（因此术语“bricking”）而不是
		一个电话。
		如果您想在实际硬件上运行自定义的AOSP构建，我建议您自己做一个像BeagleBoard xM或PandaBoard的东西。这些电路板用于修补。如果没有其他的，他们没有内置的闪存芯片，你
		可能会冒着危险。相反，这些设备上的SoC直接从SD卡启动。因此，修复损坏的图像只是将SD卡从电路板上拔下，将其连接到工作站，重新编程并将其插回电路板的问题。一些商业手
		机和设备允许您通过fastboot oem unlock命令“解锁”固件，因此您可以刻录自己的图像，同时降低设备布局的风险。然而，在这些情况下，引导程序仍然是单点故障;如果由于某种
		原因造成损坏，您最终可能会遇到一个砖块。最好的配置是你可以重新编程所有存储设备，无论你输入什么命令。
	
	adb当然可以做的不仅仅是给你一个shell，而且我鼓励你不带任何参数使用它来看它的使用输出：
	$ adb
	Android Debug Bridge version 1.0.26
	-d - directs command to the only connected USB device
			 returns an error if more than one USB device is
			 present.
	-e - directs command to the only running emulator.
			 returns an error if more than one emulator is
			 running.
	-s <serial number> - directs command to the USB device or emulator
			 with the given serial number. Overrides
		   ANDROID_SERIAL
	...
	device commands:
	adb push <local> <remote> - copy file/dir to device
	adb pull <remote> [<local>] - copy file/dir from device
	adb sync [ <directory> ] - copy host->device only if changed
	(-l means list but don't copy)
	(see 'adb help all')
	adb shell - run remote shell interactively
	adb shell <command> - run remote shell command
	adb emu <command> - run emulator console command
	...	
	例如，您可以使用adb转储主记录器缓冲区中包含的数据：
	$ adb logcat
	I/DEBUG ( 30): debuggerd: Sep 10 2011 13:44:19
	I/Netd ( 29): Netd 1.0 starting
	I/Vold ( 28): Vold 2.1 (the revenge) firing up
	D/qemud ( 38): entering main loop
	D/Vold ( 28): USB mass storage support is not enabled in the kernel
	D/Vold ( 28): usb_configuration switch is not enabled in the kernel
	D/Vold ( 28): Volume sdcard state changing -1 (Initializing) -> 0 (No-Media
	)
	D/qemud ( 38): fdhandler_accept_event: accepting on fd 9
	D/qemud ( 38): created client 0xe078 listening on fd 10
	D/qemud ( 38): client_fd_receive: attempting registration for service 'bootproperties'
	D/qemud ( 38): client_fd_receive: -> received channel id 1
	D/qemud ( 38): client_registration: registration succeeded for client 1
	I/qemu-props( 54): connected to 'boot-properties' qemud service.
	I/qemu-props( 54): receiving..
	I/qemu-props( 54): received: qemu.sf.lcd_density=160
	I/qemu-props( 54): receiving..
	I/qemu-props( 54): received: dalvik.vm.heapsize=16m
	I/qemu-props( 54): receiving..
	D/qemud ( 38): fdhandler_event: disconnect on fd 10
	I/qemu-props( 54): exiting (2 properties set).
	D/AndroidRuntime( 32):
	D/AndroidRuntime( 32): >>>>>> AndroidRuntime START com.android.internal.os.Zyg
	oteInit <<<<<<
	D/AndroidRuntime( 32): CheckJNI is ON
	I/ ( 33): ServiceManager: 0xad50
	...
	
	这对于观察关键系统组件（包括由系统服务器运行的服务）的运行时行为非常有用。
	您还可以将文件复制到设备或从设备复制文件：
	$ adb push data.txt /data/local
	1 KB/s (87 bytes in 0.043s)
	$ adb pull /proc/config.gz
	95 KB/s (7087 bytes in 0.072s)
	
	再次，考虑到Android开发的中心地位，我邀请您阅读adb的使用。我们将继续在整本书中使用它，并在第6章中对其进行了更为详细的介绍。请记住，adb可能有其怪癖。首先，很多人发现它的主机后台进程
有点薄弱。由于某种原因，它有时无法正确识别连接的设备的状态，并在尝试连接到设备时继续说明它们处于脱机状态。或者adb可能只是挂在命令行上等待设备，而设备显然是活动的并且能够接收ADB命令。
这些问题的解决方案几乎总是杀死主机端的后台进程：
		$ adb kill-server
	不要担心 - 下次发出任何adb命令时，后台进程将自动重新启动。目前还不清楚是什么导致这种行为，也许这个问题在将来的某个时候会得到解决。在此期间，请记住，如果您在使用ADB时看到一些奇怪的行为，
在调查其他潜在问题之前，通常会尝试杀死主机端守护程序。

	如上所述，我们将在第6章中更详细地讨论ADB。尽管如此，关于adb的另一个信息来源是Android的Android开发者指南中的Android Debug Bridge。正如蒂姆·布兰（Tim Bird）所建议的，您想要打印一份副本并
放在枕头下。