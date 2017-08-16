> 翻译：[neil-xiang](https://github.com/neil-xiang)

> 校对：

### 管理模拟器(Mastering the Emulator)
	如我之前所述，简单的使用模拟器对你在平台开发上大有帮助。它可以高效的模拟ARM，最新也可以模拟X86架构目标，使用最小的硬件。我们将在这里话一些时间来处理模拟器的高级功能。
与许多Android版本一样，仿真器本身就是一个非常复杂的软件。 尽管如此，我们可以通过测量几个主要功能来了解其功能。
	早些时候，我们通过键入以下命令启动了模拟器：
		$ emulator &
	但是仿真器命令也可以占用很多参数。您可以通过在命令行中添加-help标志来查看在线帮助：
		$ emulator -help
		Android Emulator usage: emulator [options] [-qemu args]
		options:
		-sysdir <dir> search for system disk images in <dir>
		-system <file> read initial system image from <file>
		-datadir <dir> write user data into <dir>
		-kernel <file> use specific emulated kernel
		-ramdisk <file> ramdisk image (default <system>/ramdisk.img
		-image <file> obsolete, use -system <file> instead
		-init-data <file> initial data image (default <system>/userdata.img
		-initdata <file> same as '-init-data <file>'
		-data <file> data image (default <datadir>/userdataqemu.
		img
		-partition-size <size> system/data partition size in MBs
		...
	一个特别有用的标志是-kernel。它允许你告诉模拟器使用另一个内核而不是在prebuilt/android-arm/kernel/目录中发现的默认预建的内核：
		$ emulator -kernel path_to_your_kernel_image/zImage
	例如，如果要使用具有模块支持的内核，则需要自行构建，因为预先构建的内核默认情况下不启用模块支持。此外，默认情况下，仿真器不会显示内核引导消息。 但是，您可以通过
-show-kernel标志来查看它们：
		$ emulator -show-kernel
		Uncompressing Linux.............................................................
		................................ done, booting the kernel.
		Initializing cgroup subsys cpu
		Linux version 2.6.29-00261-g0097074-dirty (digit@digit.mtv.corp.google.com) (gcc
		version 4.4.0 (GCC) ) #20 Wed Mar 31 09:54:02 PDT 2010
		CPU: ARM926EJ-S [41069265] revision 5 (ARMv5TEJ), cr=00093177
		CPU: VIVT data cache, VIVT instruction cache
		Machine: Goldfish
		Memory policy: ECC disabled, Data cache writeback
		Built 1 zonelists in Zone order, mobility grouping on. Total pages: 24384
		Kernel command line: qemu=1 console=ttyS0 android.checkjni=1 android.qemud=ttyS1
		android.ndns=3
		Unknown boot option `android.checkjni=1': ignoring
		Unknown boot option `android.qemud=ttyS1': ignoring
		Unknown boot option `android.ndns=3': ignoring
		PID hash table entries: 512 (order: 9, 2048 bytes)
		Console: colour dummy device 80x30
		Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
		Memory: 96MB = 96MB total
		Memory: 91548KB available (2616K code, 681K data, 104K init)
		Calibrating delay loop... 403.04 BogoMIPS (lpj=2015232)
		Mount-cache hash table entries: 512
		Initializing cgroup subsys debug
		Initializing cgroup subsys cpuacct
		Initializing cgroup subsys freezer
		CPU: Testing write buffer coherency: ok
		...
	您还可以使用-verbose标志，让仿真器打印出关于其自身执行的信息，从而允许您查看使用哪些映像文件：
		$ emulator -verbose
		emulator: found Android build root: /home/karim/android/aosp-2.3.x
		emulator: found Android build out: /home/karim/android/aosp-2.3.x/out/target/pr
		oduct/generic
		emulator: locking user data image at /home/karim/android/aosp-2.3.x/out/targ
		et/product/generic/userdata-qemu.img
		emulator: selecting default skin name 'HVGA'
		emulator: found skin-specific hardware.ini: /home/karim/android/aosp-2.3.x/sdk/e
		mulator/skins/HVGA/hardware.ini
		emulator: autoconfig: -skin HVGA
		emulator: autoconfig: -skindir /home/karim/android/aosp-2.3.x/sdk/emulator/skins
		emulator: keyset loaded from: /home/karim/.android/default.keyset
		emulator: trying to load skin file '/home/karim/android/aosp-2.3.x/sdk/emulator/
		skins/HVGA/layout'
		emulator: skin network speed: 'full'
		emulator: skin network delay: 'none'
		emulator: no SD Card image at '/home/karim/android/aosp-2.3.x/out/target/product
		/generic/sdcard.img'
		emulator: registered 'boot-properties' qemud service
		emulator: registered 'boot-properties' qemud service
		emulator: Adding boot property: 'qemu.sf.lcd_density' = '160'
		emulator: Adding boot property: 'dalvik.vm.heapsize' = '16m'
		emulator: argv[00] = "emulator"
		emulator: argv[01] = "-kernel"
		emulator: argv[02] = "/home/karim/android/aosp-2.3.x/prebuilt/android-arm/kernel
		/kernel-qemu"
		emulator: argv[03] = "-initrd"
		emulator: argv[04] = "/home/karim/android/aosp-2.3.x/out/target/product/generic/
		ramdisk.img"
		emulator: argv[05] = "-nand"
		emulator: argv[06] = "system,size=0x4200000,initfile=/home/karim/android/aosp-2.
		3.x/out/target/product/generic/system.img"
		emulator: argv[07] = "-nand"
		emulator: argv[08] = "userdata,size=0x4200000,file=/home/karim/android/aosp-2.3.
		x/out/target/product/generic/userdata-qemu.img"
		emulator: argv[09] = "-nand"
		...
	到目前为止，我已经使用了QEMU和模拟器这两个术语。实际上，仿真器命令实际上并不是QEMU：这是由Android开发团队创建的一个自定义包装。但是，您可以使用-qemu标志与模拟器
的QEMU进行交互。在该标记之后传递的任何内容都传递给QEMU，而不是仿真器包装器：
		$ emulator -qemu -h
		QEMU PC emulator version 0.10.50Android, Copyright (c) 2003-2008 Fabrice Bellard
		usage: qemu [options] [disk_image]
		'disk_image' is a raw hard image image for IDE hard disk 0
		Standard options:
		-h or -help display this help and exit
		-version display version information and exit
		-M machine select emulated machine (-M ? for list)
		-cpu cpu select CPU (-cpu ? for list)
		-smp n set the number of CPUs to 'n' [default=1]
		Mastering
		-numa node[,mem=size][,cpus=cpu[-cpu]][,nodeid=node]
		-fda/-fdb file use 'file' as floppy disk 0/1 image
		-hda/-hdb file use 'file' as IDE hard disk 0/1 image
		...
		$ emulator -qemu -...
	我们以前看到我们如何使用adb与在仿真器中运行的AOSP进行交互，我们刚刚看到我们如何使用各种选项来更改仿真器的启动方式。有趣的是，我们还可以通过电话登录来控制模拟器
在运行时的行为。启动的每个仿真器实例都在主机上分配一个端口号。再看图3-3，并检查模拟器窗口的左上角。那里的数字（这种情况下为5554）是仿真器实例正在侦听的端口号。同
时启动的下一个仿真器将获得5556，接下来的5558，等等。要访问仿真器的特殊控制台，可以使用常规telnet命令：
		$ telnet localhost 5554
		Trying 127.0.0.1...
		Connected to localhost.
		Escape character is '^]'.
		Android Console: type 'help' for a list of commands
		OK
		help
		Android console command help:
		help|h|? print a list of commands
		event simulate hardware events
		geo Geo-location commands
		gsm GSM related commands
		kill kill the emulator instance
		network manage network settings
		power power related commands
		quit|exit quit control session
		redir manage port redirections
		sms SMS related commands
		avd manager virtual device state
		window manage emulator window
		try 'help <command>' for command-specific help
		OK
	使用该控制台，您可以做一些漂亮的技巧，例如将端口从主机重定向到目标：
		redir add tcp:8080:80
		OK
		redir list
		tcp:8080 => 80
		OK
	从这里开始，在主机上访问8080的任何内容实际上都会与正在仿真Android上侦听到端口80的内容进行通话。在Android上，默认情况下没有人听到该端口，但是您可以使用例如BusyBox
的httpd在Android上运行，并以这种方式连接到它。
	模拟器还向仿真的Android公开了几个“魔术”IP。例如，IP地址10.0.2.2是工作站的127.0.0.1的别名。如果您的工作站上运行Apache，则可以打开模拟器的浏览器并输入http://10.0.2.2，
您可以浏览Apache提供的任何内容。
	有关如何操作模拟器及其各种选项的更多信息，请参阅Google Android开发人员指南的“使用Android模拟器”部分。它是针对应用程序开发人员的受众编写的，但即使您正在进行平台工作，
它仍然对您非常有用。