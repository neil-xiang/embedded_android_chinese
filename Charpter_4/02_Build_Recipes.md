> 翻译：[neil-xiang](https://github.com/neil-xiang)
> 校对：

###构建方法(Build Recipes)
	考虑到构建系统的架构和功能，让我们来看看一些最常见的，有些不太常见的构建方法。我们只会轻点使用每个方法的结果，但您应该有足够的信息来开始。
	
##默认的droid构建
	早些时候，我们经历了一些简单的make命令，但从来没有真正解释过默认目标。当你纯粹运行make时，就像你输入的一样：
			
			$ make droid
		
	droid实际上是main.mk中定义的默认目标。您通常不需要手动指定此目标。我在这里提供它的完整形式，所以你知道它存在。
	
##查看构建命令(seeing the build commands)
	当您构建AOSP时，您会注意到它并不会显示它正在运行的命令。相反，它只打印出每个步骤的摘要。如果要查看其所做的一切，例如gcc命令行，请将showcommands目标添加到命令行：
	
			$ make showcommands
			
	说明我在上一节中所解释的内容，与以下内容相同：
			
			$ make droid showcommands
			
	正如你在使用这种情况时会很快注意到的，它会产生大量的输出，因此很难跟踪。但是，如果要分析用于构建AOSP的实际命令，则可能希望将标准输出和标准错误保存到文件中：
	
			$ make showcommands > aosp-build-stdout 2> aosp-build-stderr
			
	您还可以执行此操作将所有输出合并到单个文件中：
	
			$ make showcommands 2>&1 | tell build.log
			
	有些还说，他们更喜欢使用nohup命令：
	
			$ nohup make showcommands
			
	
##构建适用于Linux和Mac OS的SDK(Building the SDK for Linux and Mac OS)

	官方的Android SDK可以在http://developer.android.com上找到。然而，您可以使用AOSP构建自己的SDK，例如，如果扩展了核心API以展示新功能，并希望将结果分发给开发人员，
以便他们可以从新的API中受益。为此，您需要选择一个特殊的组合：

		$ . build/envsetup.sh
		$ lunch sdk-eng
		$ make sdk
	
	一旦这些命令执行完，SDK会在out/host/linux-x86/sdk/(linux下构建)和out/host/darwin-x86/sdk/(Mac下构建)。将有两个副本，一个ZIP文件，非常像在http://developer.andr-
oid.com上分发的一个，一个未压缩并可以使用。

	假设您已经使用http://developer.android.com上的说明配置Eclipse进行Android开发，您需要执行两个额外的步骤来使用您新建的SDK。首先，您需要告诉Eclipse新SDK的位置。要
实现这个目的，请转到窗口→首选项→Android，在“SDK位置”框中输入新SDK的路径，然后单击“确定”。另外，由于在撰写本文时并不完全清楚的原因，您还需要转到Window→Android SDK
Manager，取消选择所有可能选择的项目，除了前两个在“工具”下，然后单击“安装2个软件包...”完成之后，您将能够使用新的SDK创建新项目，并访问您在其中公开的任何新API。如果
您不这样做第二步，您将能够创建新的Android项目，但没有一个将正确解析Java库，因此将永远不会构建。

##构建适用于Windows的SDK(Building the SDK for Windows)

	构建Windows SDK的说明与Linux和Mac OS略有不同：
	
			$ . build/envsetup.sh
			$ lunch sdk-eng
			$ make win_sdk
			
	结果输出将在/host/windows/sdk/中。
	
##构建CTS(Building the CTS)

	如果你想建立CTS，你不需要使用envsetup.sh或lunch。你可以直接输入：
			
			$ make cts
			
	cts命令包含自己的在线帮助。以下是Android2.3/Gingerbread的相应示例输出：
			
			$ cd out/host/linux-x86/bin/
			$ ./cts
			$ cts_host > help
			$ cts_host > ls --plan
			
	一旦你有一个目标运行，如模拟器，你可以启动测试套件，它将使用adb来运行目标测试：
			
			$ ./cts start --plan CTS
			
			
##构建NDK(Building the NDK)
	
	如前所述，NDK有自己的独立构建系统，具有自己的安装和帮助系统，您可以这样调用：
	
			$ cd ndk/build/tools
			$ export ANDROID_NDK_ROOT=aosp-root/ndk
			$ ./make-release --help
	
	当您准备构建NDK时，可以按以下方式调用make-release，并且看到其相当强调的警告：
			$ ./make-release
			IMPORTANT WARNING !!
			This script is used to generate an NDK release package from scratch
			for the following host platforms: linux-x86
			This process is EXTREMELY LONG and may take SEVERAL HOURS on a dual-core
			machine. If you plan to do that often, please read docs/DEVELOPMENT.TXT
			that provides instructions on how to do that more easily.
			Are you sure you want to do that [y/N]
			y
			Downloading toolchain sources...
			...
			
##更新API(Updating the API)
	
	构建系统具有保护措施来防止你修改了AOSP的核心API。如果你这样做了，默认情况下，构建将失败，并显示如下警告：
	
		******************************
		You have tried to change the API from what has been previously approved.
		To make these errors go away, you have two choices:
		1) You can add "@hide" javadoc comments to the methods, etc. listed in the
		errors above.
		2) You can update current.xml by executing the following command:
		make update-api
		To submit the revised current.xml to the main Android repository,
		you will need approval.
		******************************
		make: *** [out/target/common/obj/PACKAGING/checkapi-current-timestamp] Error 38
		make: *** Waiting for unfinished jobs....
		
	如错误信息所示，为了继续构建，您需要执行以下操作：
		$ make update-api
		...
		Install: out/host/linux-x86/framework/apicheck.jar
		Install: out/host/linux-x86/framework/clearsilver.jar
		Install: out/host/linux-x86/framework/droiddoc.jar
		Install: out/host/linux-x86/lib/libneo_util.so
		Install: out/host/linux-x86/lib/libneo_cs.so
		Install: out/host/linux-x86/lib/libneo_cgi.so
		Install: out/host/linux-x86/lib/libclearsilver-jni.so
		Copying: out/target/common/obj/JAVA_LIBRARIES/core_intermediates/emma_out/lib/cl
		asses-jarjar.jar
		Install: out/host/linux-x86/framework/dx.jar
		Install: out/host/linux-x86/bin/dx
		Install: out/host/linux-x86/bin/aapt
		Copying: out/target/common/obj/JAVA_LIBRARIES/bouncycastle_intermediates/emma_ou
		t/lib/classes-jarjar.jar
		Copying: out/target/common/obj/JAVA_LIBRARIES/ext_intermediates/emma_out/lib/cla
		sses-jarjar.jar
		Install: out/host/linux-x86/bin/aidl
		Copying: out/target/common/obj/JAVA_LIBRARIES/core-junit_intermediates/emma_out/
		lib/classes-jarjar.jar
		Copying: out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/emma_out/l
		ib/classes-jarjar.jar
		Copying current.xml
		
	下次开始make时，您不会再收到有关API更改的错误信息。显然，在这一点上，您不再与官方API兼容，因此不太可能被Google认证为“Android”设备。
	
##构建一个独立模块(Building a Single Module)

	到目前为止，我们已经研究了构建整个树。您还可以构建单个模块。以下是如何要求构建系统构建Launcher2模块（即主屏幕）：
	
			$ make Launcher2
	
	您也可以单独clean模块：
	
			$ make clean-Launcher2
			
	如果要强制构建系统重新生成系统映像以包含更新的模块，则可以将snod目标添加到命令行：
			
			$ make Launcher2 snod
			============================================
			PLATFORM_VERSION_CODENAME=REL
			PLATFORM_VERSION=2.3.4
			TARGET_PRODUCT=generic
			...
			target Package: Launcher2 (out/target/product/generic/obj/APPS/Launcher2_interme
			diates/package.apk)
			'out/target/common/obj/APPS/Launcher2_intermediates//classes.dex' as 'classes.d
			ex'...
			Install: out/target/product/generic/system/app/Launcher2.apk
			Install: out/host/linux-x86/bin/mkyaffs2image
			make snod: ignoring dependencies
			Target system fs image: out/target/product/generic/system.img
			
	
##脱离树的构建(Build Out of Tree)
	如果您希望针对AOSP及其仿生库构建代码，但不希望将其纳入AOSP，您可以使用如以下类似的makefile来完成工作：
	
			# Paths and settings
			TARGET_PRODUCT = generic
			ANDROID_ROOT = /home/karim/android/aosp-2.3.x
			BIONIC_LIBC = $(ANDROID_ROOT)/bionic/libc
			PRODUCT_OUT = $(ANDROID_ROOT)/out/target/product/$(TARGET_PRODUCT)
			CROSS_COMPILE = \
			$(ANDROID_ROOT)/prebuilt/linux-x86/toolchain/arm-eabi-4.4.3/bin/arm-eabi-
			# Tool names
			AS = $(CROSS_COMPILE)as
			AR = $(CROSS_COMPILE)ar
			CC = $(CROSS_COMPILE)gcc
			CPP = $(CC) -E
			LD = $(CROSS_COMPILE)ld
			NM = $(CROSS_COMPILE)nm
			OBJCOPY = $(CROSS_COMPILE)objcopy
			OBJDUMP = $(CROSS_COMPILE)objdump
			RANLIB = $(CROSS_COMPILE)ranlib
			READELF = $(CROSS_COMPILE)readelf
			SIZE = $(CROSS_COMPILE)size
			STRINGS = $(CROSS_COMPILE)strings
			STRIP = $(CROSS_COMPILE)strip
			export AS AR CC CPP LD NM OBJCOPY OBJDUMP RANLIB READELF \
			SIZE STRINGS STRIP
			# Build settings
			CFLAGS = -O2 -Wall -fno-short-enums
			HEADER_OPS = -I$(BIONIC_LIBC)/arch-arm/include \
			-I$(BIONIC_LIBC)/kernel/common \
			-I$(BIONIC_LIBC)/kernel/arch-arm
			LDFLAGS = -nostdlib -Wl,-dynamic-linker,/system/bin/linker \
			$(PRODUCT_OUT)/obj/lib/crtbegin_dynamic.o \
			$(PRODUCT_OUT)/obj/lib/crtend_android.o \
			-L$(PRODUCT_OUT)/obj/lib -lc -ldl
			# Installation variables
			EXEC_NAME = example-app
			INSTALL = install
			INSTALL_DIR = $(PRODUCT_OUT)/system/bin
			# Files needed for the build
			OBJS = example-app.o
			# Make rules
			all: example-app
			.c.o:
			$(CC) $(CFLAGS) $(HEADER_OPS) -c $<
			example-app: ${OBJS}
			$(CC) -o $(EXEC_NAME) ${OBJS} $(LDFLAGS)
			install: example-app
			test -d $(INSTALL_DIR) || $(INSTALL) -d -m 755 $(INSTALL_DIR)
			$(INSTALL) -m 755 $(EXEC_NAME) $(INSTALL_DIR)
			clean:
			rm -f *.o $(EXEC_NAME) core
			distclean:
			rm -f *~
			rm -f *.o $(EXEC_NAME) core
			
	在这种情况下，您不需要关心envsetup.sh或lunch。你可以直接输入'魔法咒语'：
			
			$ make
			
	显然，这不会将您的二进制文件添加到AOSP生成的任何镜像中。即使install的目标在将目标文件系统从NFS上卸载才有效，而且有效值只在debug是才是有效的，这是makefile被认为
是有用的。在某种程度上，也可以认为使用这样一个makefile实际上是适得其反的，因为它远远超过了将这个代码作为AOSP的模块部分添加时产生的相当于Android.mk的复杂性。
	不过，这种黑客可以有其用途。例如，在某些情况下，修改由相当大的代码库使用的常规构建系统可能有意义，以在其外部的AOSP上构建该项目;另一种方法是将项目复制到AOSP中，
并创建Android.mk文件来重现其原始常规构建系统的机制，这可能是本身的一个实质性的工作。

##递归构建(Building Recursively,In-Tree)

	如果你真的想要，你可以自己修改一个makefile来构建一个基于递归makefile的组件，而不是使用Android.mk文件来重现相同的功能，如上一节所述。例如，附录E中提到的几个AOSP
分支将内核源包括在AOSP的顶层，并修改AOSP的主makefile以调用内核的现有构建系统。
	这是另一个例子，由Linaro的BernhardRosenkränzer创建一个Android.mk，以便使用其原始的构建文件来构建基于GNU自动工具类脚本的ffmpeg：