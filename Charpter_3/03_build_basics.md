> 翻译：[koffuxu](https://github.com/koffuxu)

> 校对：

### 构建基础(Build Basics)
	现在我们已经下载好了AOSP，那我们就产生一个里面有什么的想法，所以，我们让其运行起来。但最重要的是我们首先要能够构建它。我们需要确认在我们的Ubuntu上已经安装了必要的包。接下来的操作是
基于Ubuntu 11.04，假设我们要构建Android2.3。就算你是使用比这新或者旧版本的基于Debian的Linux改造版本，这些操作应该也是相近的。（对其它那些能够编译AOSP的系统参考[在非Ubuntu系统或者虚拟
机上编译]）。像我之前提到的那样，参考Google的”Initializing a Build Environment“来得到在最新的Ubuntu版本上构建最新的AOSP版本最新的包。

# 编译系统设置

	首先，在我们的开发系统中安装一些基础的包。如果你在其它开发的过程中安装了一些这种包，OK，没问题。Ubuntu系统包管理系统将忽略这些软件包安装。
			**注意，在下面的命令中由于书的宽度的限制命令被分成了几行。在shell中行的末尾使用'\'强制shell将另一行重新开始让你有机会继续输入你的命令。因此，你必须在接下来的命令末尾输入’\‘，但
				是后面的行开始的’>‘不是你需要输入的，它由shell自动插入。本书中的其它命令基于相同的原因使用同样的技巧。
				
	$ sudo apt-get install bison flex gperf git-core gnupg zip tofrodos \
	> build-essential g++-multilib libc6-dev libc6-dev-i386 ia32-libs mingw32 \
	> zlib1g-dev lib32z1-dev x11proto-core-dev libx11-dev \
	> lib32readline5-dev libgl1-mesa-dev lib32ncurses5-dev

你也许也需要修正一些符号链接：

	$ sudo ln -s /usr/lib32/libstdc++.so.6 /usr/lib32/libstdc++.so
	$ sudo ln -s /usr/lib32/libz.so.1 /usr/lib32/libz.so

	最后，你需要安装Sun公司的JDK；使用OpenJDK与AOSP“官方”是不正式的，尽管有些人可以成功使用它，但是gcj不会这样做。在Ubuntu中，您曾经通过使用以下命令序列来获取JDK：

	$ sudo add-apt-repository "deb http://archive.canonical.com/ natty partner"
	$ sudo apt-get update
	$ sudo apt-get install sun-java6-jdk
	
	不幸的是，Canonical（Ubuntu背后的公司）和Oracle之间似乎有一些分歧，并且这些指令在撰写本文时不再工作。相反，您应该参考Ubuntu的指导，让JDK V6在你的主机上运行。请注意，在撰写本文时，版
本7在AOSP上不能工作。从本质上讲，Ubuntu的说明解释说，您需要从Oracle的站点获取JDK二进制文件并进行安装。这是一个稍微修改版本的当前发布的指令，您可能需要适应最新版本的JDK:

	$ chmod u+x jdk-6u38-linux-x64.bin
	$ ./jdk-6u38-linux-x64.bin
	$ sudo mkdir -p /usr/lib/jvm
	$ sudo mv jdk1.6.0_38 /usr/lib/jvm/
	$ sudo update-alternatives --install "/usr/bin/java" "java" \
	> "/usr/lib/jvm/jdk1.6.0_38/bin/java" 1
	$ sudo update-alternatives --install "/usr/bin/javac" "javac" \
	> "/usr/lib/jvm/jdk1.6.0_38/bin/javac" 1
	$ sudo update-alternatives --install "/usr/bin/javah" "javah" \
	> "/usr/lib/jvm/jdk1.6.0_38/bin/javah" 1
	$ sudo update-alternatives --install "/usr/bin/javadoc" "javadoc" \
	> "/usr/lib/jvm/jdk1.6.0_38/bin/javadoc" 1
	$ sudo update-alternatives --install "/usr/bin/jar" "jar" \
	> "/usr/lib/jvm/jdk1.6.0_38/bin/jar" 1
	
	现在你可以运行下面的命令选择你刚刚安装的java的版本
	$ sudo update-alternatives --config java
	There are 2 choices for the alternative java (providing /usr/bin/java).
	Selection Path Priority Status
	---------------------------------------------------------
	* 0 /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java 1061 auto mode
	1 /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java 1061 manual mode
	2 /usr/lib/jvm/jdk1.6.0_38/bin/java 1 manual mode
	Press enter to keep the current choice[*], or type selection number: 2
	$ sudo update-alternatives --display java
	java - manual mode
	link currently points to /usr/lib/jvm/jdk1.6.0_38/bin/java
	...
	$ sudo update-alternatives --config javac
	...
	$ sudo update-alternatives --config javah
	...
	$ sudo update-alternatives --config javadoc
	...
	$ sudo update-alternatives --config jar
	...
	
	如你所见，Oracle的JDK和OpenJDK可以在同一个Ubuntu安装系统中共存。你只需要确认默认的指向你需要的正确的JDK就行了。上述的说明让你安装Oracle的JDK系统并通过更改命令使
安装包中的二进制文件来替换在Ubuntu系统中的默认软件。没有什么可以阻止你将Oracle的JDK安装在你的根目录下并更改 PATH 变量指向运行Oracle的安装文件解压后的 bin/ 目录
你的系统现在可以准备编译Android了。明显地，在你的以后编译Android的时候，不需要再安装这些包。在你每台Android开发系统只需要设置一次。

## 编译Android

现在我们开始编译Andorid，首先进入下载好源码的目录，设置编译系统：


```

	$ cd ~/android/aosp-2.3.x
	$ . build/envsetup.sh
	including device/acme/coyotepad/vendorsetup.sh
	including device/htc/passion/vendorsetup.sh
	including device/samsung/crespo4g/vendorsetup.sh
	including device/samsung/crespo/vendorsetup.sh
	$ lunch
	You're building on Linux
	Lunch menu... pick a combo:
     1. generic-eng
     2. simulator
     3. full_passion-userdebug
     4. full_crespo4g-userdebug
     5. full_crespo-userdebug
	Which would you like? [generic-eng] ENTER
	============================================
	PLATFORM_VERSION_CODENAME=REL
	PLATFORM_VERSION=2.3.4
	TARGET_PRODUCT=generic
	TARGET_BUILD_VARIANT=eng
	TARGET_SIMULATOR=false
	TARGET_BUILD_TYPE=release
	TARGET_BUILD_APPS=
	TARGET_ARCH=arm
	HOST_ARCH=x86
	HOST_OS=linux
	HOST_BUILD_TYPE=release
	BUILD_ID=GINGERBREAD
	============================================
	对于Android4.2/Jelly Bean来说，在Ubuntu 12.04上同样的操作可能出现下列的结果
	$ cd ~/android/aosp-4.2
	$ . build/envsetup.sh
	including device/asus/grouper/vendorsetup.sh
	including device/asus/tilapia/vendorsetup.sh
	including device/generic/armv7-a-neon/vendorsetup.sh
	including device/generic/armv7-a/vendorsetup.sh
	including device/generic/mips/vendorsetup.sh
	including device/generic/x86/vendorsetup.sh
	including device/lge/mako/vendorsetup.sh
	including device/samsung/maguro/vendorsetup.sh
	including device/samsung/manta/vendorsetup.sh
	including device/samsung/toroplus/vendorsetup.sh
	including device/samsung/toro/vendorsetup.sh
	including device/ti/panda/vendorsetup.sh
	including sdk/bash_completion/adb.bash
	$ lunch
	
	You're building on Linux
	Lunch menu... pick a combo:
		1. full-eng
		2. full_x86-eng
		3. vbox_x86-eng
		4. full_mips-eng
		5. full_grouper-userdebug
		6. full_tilapia-userdebug
		7. mini_armv7a_neon-userdebug
		8. mini_armv7a-userdebug
		9. mini_mips-userdebug
		10. mini_x86-userdebug
		11. full_mako-userdebug
		12. full_maguro-userdebug
		13. full_manta-userdebug
		14. full_toroplus-userdebug
		15. full_toro-userdebug
		16. full_panda-userdebug
		
		Which would you like? [full-eng] ENTER
	============================================
	PLATFORM_VERSION_CODENAME=REL
	PLATFORM_VERSION=4.2
	TARGET_PRODUCT=full
	TARGET_BUILD_VARIANT=eng
	TARGET_BUILD_TYPE=release
	TARGET_BUILD_APPS=
	TARGET_ARCH=arm
	TARGET_ARCH_VARIANT=armv7-a
	HOST_ARCH=x86
	HOST_OS=linux
	HOST_OS_EXTRA=Linux-3.2.0-35-generic-x86_64-with-Ubuntu-12.04-precise
	HOST_BUILD_TYPE=release
	BUILD_ID=JOP40C
	OUT_DIR=out
	============================================

	注意，在两种情况下，我们都输入‘.'和‘空格’，这是在当前终端强制shell运行`envsetup.sh`脚本。如果我们只是运行这个脚本，shell会产生一个新的shell并在新的shell上运行这个
脚本。当'envsetup.sh'定义了新的命令就没有用了，比如‘lunch’，并却在接下来的build中需要设置环境变量
	我们会在稍后详细的解释envsetup.sh和lunch。现在，我们注意在Android2.3/Gingerbread中'generic-eng'和Android4.2/JellyBean中的‘full-eng’意思是我们在配置构建系统来生成
一个在Android模拟器上运行的镜像。这和开发人员在工作站上运行QEMU模拟软件来测试他们的使用SDK开发的app是一样的。在Android的模拟器上，我们运行的是我们自己的镜像而不是
和SDK一起的一个缺省app。这也被Android开发团队在开发Android时还没有具体的设备时进行模拟。所以虽然它不是真正的硬件，因此绝对不是一个完美的目标，但仍然足以覆盖我们需要
覆盖的大部分情形。如果你知道你特定的目标，你可以修改在本书中接下来找到的说明，也许需要参考‘Building Embedded Linux Systems’，来得到你定制化的Android镜像导入到你的硬
件来启动它们。
	现在环境已经设置好了，我们就能真正的开始编译Android了。
	
	$ make -j16
	============================================
	PLATFORM_VERSION_CODENAME=REL
	PLATFORM_VERSION=2.3.4
	TARGET_PRODUCT=generic
	TARGET_BUILD_VARIANT=eng
	TARGET_SIMULATOR=false
	TARGET_BUILD_TYPE=release
	TARGET_BUILD_APPS=
	TARGET_ARCH=arm
	HOST_ARCH=x86
	HOST_OS=linux
	HOST_BUILD_TYPE=release
	BUILD_ID=GINGERBREAD
	============================================
	Checking build tools versions...
	find: `frameworks/base/frameworks/base/docs/html': No such file or directory
	find: `out/target/common/docs/gen': No such file or directory
	find: `frameworks/base/frameworks/base/docs/html': No such file or directory
	find: `out/target/common/docs/gen': No such file or directory
	find: `frameworks/base/frameworks/base/docs/html': No such file or directory
	find: `out/target/common/docs/gen': No such file or directory
	find: `frameworks/base/frameworks/base/docs/html': No such file or directory
	find: `out/target/common/docs/gen': No such file or directory
	find: `frameworks/base/frameworks/base/docs/html': No such file or directory
	find: `out/target/common/docs/gen': No such file or directory
	host Java: apicheck (out/host/common/obj/JAVA_LIBRARIES/apicheck_intermediates/c
	lasses)
	Header: out/host/linux-x86/obj/include/libexpat/expat.h
	Header: out/host/linux-x86/obj/include/libexpat/expat_external.h
	Header: out/target/product/generic/obj/include/libexpat/expat.h
	Header: out/target/product/generic/obj/include/libexpat/expat_external.h
	Header: out/host/linux-x86/obj/include/libpng/png.h
	Header: out/host/linux-x86/obj/include/libpng/pngconf.h
	Header: out/host/linux-x86/obj/include/libpng/pngusr.h
	Header: out/target/product/generic/obj/include/libpng/png.h
	Header: out/target/product/generic/obj/include/libpng/pngconf.h
	Header: out/target/product/generic/obj/include/libpng/pngusr.h
	Header: out/target/product/generic/obj/include/libwpa_client/wpa_ctrl.h
	Header: out/target/product/generic/obj/include/libsonivox/eas_types.h
	Header: out/target/product/generic/obj/include/libsonivox/eas.h
	Header: out/target/product/generic/obj/include/libsonivox/eas_reverb.h
	Header: out/target/product/generic/obj/include/libsonivox/jet.h
	Header: out/target/product/generic/obj/include/libsonivox/ARM_synth_constants_gn
	u.inc
	host Java: clearsilver (out/host/common/obj/JAVA_LIBRARIES/clearsilver_intermedi
	ates/classes)
	target Java: core (out/target/common/obj/JAVA_LIBRARIES/core_intermediates/class
	es)
	host Java: dx (out/host/common/obj/JAVA_LIBRARIES/dx_intermediates/classes)
	Notice file: frameworks/base/libs/utils/NOTICE -- out/host/linux-x86/obj/NOTICE_
	FILES/src//lib/libutils.a.txt
	Notice file: system/core/libcutils/NOTICE -- out/host/linux-x86/obj/NOTICE_FILES
	/src//lib/libcutils.a.txt
	...

	现在，你可以去吃点零食，或者看看今天晚上的曲棍球比赛。需要引起你注意的是，你编译的时间是依赖于你的系统性能。在一台启用超线程的四核CORE i7的笔记本电脑中，装备8G
内存，执行这个命令后大概20分钟编译完Android2.3/Gingerbread，大约80分钟编译完Android4.2/Jelly Bean。在一台老式笔记本电脑上，双核 Centro 2的Inter处理器，拥有2G内存，
执行`*make -j4*`来编译同一套Android2.3/Gingerbread的话大概要花费一个小时。我不会再这样一台机器上来试图编译Android4.2/Jelly Bean的。
注意在`*make*`之后接的`*-j*`参数是指定并行多少个任务。有一种说法是这个最佳值是你的处理器核的2倍，那就是我们刚才指定的。另一种说法是你处理器核的数量再加2。按照这种
说法，我们应该使用10和4而不是16和4。 
	通常来说，AOSP是编译软件中非常重的一块。我强烈建议你使用你手上最强力的系统-没有任何限制。拥有大量的内存也是值得推荐的。事实上，如果整个AOSP都可以在内存中保存的文
件系统缓存中，这样可以最小化编译时间。你也可以使用固态硬盘代替机械硬盘，这样也可以大大的减少AOSP的编译时间。


		** 在非Ubuntu系统和虚拟机上编译
		我经常被问到在虚拟机上编译AOSP的问题；大部分是因为开发团队，或者IT部门的标准工作环境是Windows。然后这项工作是整理我自己的图片来完成，你的结果也许有点不一样。在虚拟机上编译与在真实的机器上编译通常要花费多两倍时间。所以，如果你有许多工作要在AOSP上完成，我强烈建议你在真实的机器上编译。是的，你手头需要一台Linux环境的电脑。
		越来越多的开发者更喜欢在Mac OS X上开发，而不是Linux或者Windows。包括Google自己内部员工。因此，官方在[http://source.android.com]()的编译说明是基于Mac的描述。这些说明倾向于Mac OS更新后被打断，对于基于Mac开发者，幸运的是，他们数量众多，而且是热心的。因此，在Mac OS更新之后，你在网站或者在众多的Google小组里面最终会找到怎样编译AOSP，更新的说明。这有一篇文章是说明怎样在Mac OS X Lion上编译Gingerbread：[Building Gingerbread on OS X Lion]()。记住，我在(第一章)[]已经提到，Google自己的Android是在基于Ubuntu上顺利编译。如果你选择在Mac OS上编译，那么你将一直扮演一个对接角色。最坏的情况，你可以在Windows使用VM虚拟机这种情况。
		如果你选择走VM路线，确保你的配置是VM虚拟机使用你系统具备的多个CPU。我之前看到的许多BIOS设置中，“使能CPU指令集允许多CPU虚拟化”这项是禁止的。例如，当这些指令集禁的时候，如果你申请多个CPU时，在*VitrualBox*平台会产生*obscur错误*。你必须进入BIOS，并且使能这个选项，授权你的虚拟机使用多个CPU。
		
	这还有些编译的事需要考虑到。首先，注意在打印编译配置和打印真正的编译（这里会打印`“host Java: apicheck (out/host/common/o...”`）之间，除了打印`“No such file or directory”`这个之外，将持续一段时间没有任何打印。我在后面将解决这种延时。现在要说的就是，找出怎样编译AOSP的每一部分，在整个编译的过程中。

	同样也要注意，你将看到一些警告声明。这些是正常的，不是与维护软件质量那么地位重要，但是在整个Android的编译的过程是常见了。这对最终产品的编译不会有什么影响的。所以，与我们中最好的软件工程师相反的是，我必须建议你完全忽略这些警告，专注于解决错误上来。当然，这些警告要防止它们来自你的添加部分上。也就是说，确保你不能新加警告。
