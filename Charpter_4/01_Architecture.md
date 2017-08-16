> 翻译：[neil-xiang](https://github.com/neil-xiang)
> 校对：

###体系结构(Architecture)
	
	如图4-1所示，构建系统的入口点是build/core/目录中的main.mk文件，它通过顶层makefile调用，如前所述。build/core/目录实际上包含了大量的构建系统，我们将从那里介绍关键
文件。请再次记住，Android构建系统将所有内容都放到单个makefile中; 它不是递归的。因此，您看到的每个.mk文件最终都将成为单个巨大makefile的一部分，其中包含构建系统中所
有部分的规则。

##配置(Configuration)
	构建系统的首要任务之一是通过包含config.mk来引入构建配置。可以通过使用envsetup.sh和lunch命令或在顶层目录中提供buildspec.mk文件来配置构建。在这两种情况下，需要设置
以下一些变量。
	TARGET_PRODUCT
		Android的构建风格。例如，每个风格可以包括一组不同的应用程序或区域设置或构建树的不同部分。看看在build/target/product/中的AndroidProducts.mk文件包含的各种单一产品
		的.mk文件，如Android2.3/Gingerbread下的device/samsung/crespo/, and device/htc/passion/目录。在Android4.2/Jelly Bean的情况下，请查看device/asus/biter/和device/
		samsung/amgnuro/而不是Crespo和Passion。值包括以下内容：
		
		generic
			“vanilla”的种类，你可以得到的AOSP最基本的部分。
		full
			“all dressed”的类型，大多数应用程序和主要语言环境启用。
		full_crespo
			与‘full’相同，但是是为Crespo（Samsung Nexus S）准备的。
		full_grouper
			与‘full’相同，但是是为Grouper (Asus Nexus 7)准备的。
		sim
			Android模拟器.即使这在Android2.3/Gingerbread中可用，但此目标已被删除，并不在Android4.2/Jelly Bean中。
		sdk
			SDK
			
	TARGET_BUILD_VARIANT
		选择要安装的模块。每个模块应该在其Android.mk中设置一个LOCAL_MODULE_TAGS变量,至少包含以下之一：user,debug,eng,tests,optional,samples.通过选择该变量，您将告诉构
		建系统应该包括哪些模块子集-唯一的例外是这些规则不适用的包（即生成.apk文件的模块）。特别来说：
		
		eng
			包括标记为user，debug或eng的所有模块。
		userdebug
			包括标记为用户和调试的两个模块。
		user
			仅包含标记为用户的模块
			
	TARGET_BUILD_TYPE
		说明是否使用特殊构建标志，或在代码中定义DEBUG变量。这里可能的值是‘release’或'debug'。最值得注意的是，frameworks/base/Android.mk文件在frameworks/base/core/
		config/debug和frameworks/base/core/config/ndebug之间选择，这取决于该变量是否设置为'debug'。前者导致ConfigBuildFlags。DEBUG将Java常量设置为true，而后者会将
		其设置为false。例如，部分系统服务中的某些代码以DEBUG为条件。通常，TARGET_BUILD_TYPE设置为'release'。
		
	TARGET_TOOLS_PREFIX
		默认情况下，构建系统将使用其在prebuilt/目录(Android4.2下是prebuilts/目录)之下附带的交叉开发工具链之一。但是，如果您希望使用其他工具链，则可以将此值设置为指
		向其位置。
		
	OUT_DIR
		默认情况下，构建系统将所有构建输出都放入out/目录中。你可以使用此变量提供备用的输出目录。
		
	BUILD_ENV_SEQUENCE_NUMBER
		如果您使用模板build/buildspec.mk.default来创建自己的build spec.mk文件，则该值将被正确设置。但是，如果您使用较早的AOSP版本创建buildspec.mk，并尝试在将来的AOSP版
		本中使用它，其中包含对其构建系统的重要更改，因此，一个不同的值，这个变量将作为一个安全网。这将导致构建系统通知您，您的buildspec.mk文件与您的构建系统不匹配。
		
	除了选择要建立的AOSP的哪些部分以及与其建立的哪些选项之外，构建系统还需要了解其建立的目标。这是通过一个BoardConfig.mk文件提供的，这将指定要提供给内核的命令行，应该
加载内核的基地址，或最适合板卡CPU（TARGET_ARCH_VARIANT）的指令集版本。看看build/target/board/一组每个目标的目录，每个目录都包含一个BoardConfig.mk文件。还可以看看AOSP
中包含的各种device/*/TARGET_DEVICE/BoardConfig.mk文件。后者比前者更丰富，因为它们包含更多的硬件特定信息。设备名称（即TARGET_DEVICE）是从为配置中的TARGET_PRODUCT集提供
的product.mk文件中指定的PRODUCT_DEVICE派生的。例如，在Android2.3/Gingerbread中，device/samsung/crespo/AndroidProducts.mk包括device/samsung/crespo/full_crespo.mk，将
PRODUCT_DEVICE设置为crespo。因此，构建系统在device/*/crespo/中找到一个BoardConfig.mk，在该位置恰好有一个。Android4.2/Jelly Bean中的device/asus/grouper/full_grouper.mk
中的PRODUCT_DEVICE设置为'grouper'，从而将构建系统指向device/*/grouper/BoardConfig.mk。
	关于配置的最后一块谜题是用于构建Android的CPU特定选项。对于ARM，这些包含在build/core/combo/arch/arm/armv*.mk中，TARGET_ARCH_VARIANT确定要使用的实际文件。每个文件列出
用于构建C / C ++文件的特定于CPU的交叉编译器和交叉链接器标志。它们还包含许多ARCH_ARM_HAVE_ *变量，使得AOSP的其他部分能够基于目标CPU中是否找到给定的ARM功能有条件地构建代码。

##envsetup.sh
	现在，您了解构建系统需要的配置输入种类，我们可以更详细地讨论envsetup.sh的作用。顾名思义，envsetup.sh实际上是为Android设置一个构建环境。但它只是部分工作。主要地，它定义
了一系列可用于任何类型的AOSP工作的shell命令：
	$ cd ~/android/aosp-2.3.x
	$ . build/envsetup.sh
	$ help
	Invoke ". build/envsetup.sh" from your shell to add the following functions to
	your environment:
	- croot: Changes directory to the top of the tree.
	- m: Makes from the top of the tree.
	- mm: Builds all of the modules in the current directory.
	- mmm: Builds all of the modules in the supplied directories.
	- cgrep: Greps on all local C/C++ files.
	- jgrep: Greps on all local Java files.
	- resgrep: Greps on all local res/*.xml files.
	- godir: Go to the directory containing a file.
	Look at the source to view more functions. The complete list is:
	add_lunch_combo cgrep check_product check_variant choosecombo chooseproduct choo
	setype choosevariant cproj croot findmakefile gdbclient get_abs_build_var getbug
	reports get_build_var getprebuilt gettop godir help isviewserverstarted jgrep lu
	nch m mm mmm pgrep pid printconfig print_lunch_menu resgrep runhat runtest set_j
	ava_home setpaths set_sequence_number set_stuff_for_environment settitle smokete
	st startviewserver stopviewserver systemstack tapas tracedmdump
	
	在4.2/Jelly Bean中，hmm已经取代了help，并且提供给您的命令集已经扩展了：
	$ cd ~/android/aosp-4.2
	$ . build/envsetup.sh
	$ hmm
	Invoke ". build/envsetup.sh" from your shell to add the following functions to y
	our environment:
	- lunch: lunch <product_name>-<build_variant>
	- tapas: tapas [<App1> <App2> ...] [arm|x86|mips] [eng|userdebug|user]
	- croot: Changes directory to the top of the tree.
	- m: Makes from the top of the tree.
	- mm: Builds all of the modules in the current directory.
	- mmm: Builds all of the modules in the supplied directories.
	- cgrep: Greps on all local C/C++ files.
	- jgrep: Greps on all local Java files.
	- resgrep: Greps on all local res/*.xml files.
	- godir: Go to the directory containing a file.
	Look at the source to view more functions. The complete list is:
	addcompletions add_lunch_combo cgrep check_product check_variant choosecombo cho
	oseproduct choosetype choosevariant cproj croot findmakefile gdbclient get_abs_b
	uild_var getbugreports get_build_var getlastscreenshot getprebuilt getscreenshot
	path getsdcardpath gettargetarch gettop godir hmm isviewserverstarted jgrep key_
	back key_home key_menu lunch _lunch m mm mmm pid printconfig print_lunch_menu re
	sgrep runhat runtest set_java_home setpaths set_sequence_number set_stuff_for_en
	vironment settitle smoketest startviewserver stopviewserver systemstack tapas tr
	acedmdump
	
	你可能会发现croot和godir命令对于遍历树很有用。鉴于使用Java及其要求将包存储在与相应的完全限定包名称的每个子部分具有相同层次结构的目录树中的要求相当深入。例如，
com.foo.bar包的文件部分必须存储在com/foo/bar/目录下。因此，在AOSP的顶级目录下面找到自己的7到10个目录并不罕见，并且像cd ../../../ ...一样以返回到树的上部。快速
变得乏味.
	m和mm也是非常有用的，因为它们允许您分别从顶层构建，无论您在哪里，或只是构建在当前目录中找到的模块。例如，如果您对Launcher进行了修改，并且在packages/apps/Lau-
ncher2中，则可以通过键入mm而不是cd回到顶级并键入make来重建该模块。请注意，mm不会重建整个树，因此即使依赖模块已更改，也不会重新生成AOSP镜像。然而m会做那个。尽管
如此，mm可以用来测试您的本地更改是否打破构建，直到您准备好重新生成完整的AOSP。
	虽然在线帮助没有提到'lunch'，它是由envsetup.sh定义的命令之一。如果你使用‘lunch’是没有任何参数，它会显示一些潜在的选择。这是2.3 / Gingerbread的列表：
		$ lunch
		You're building on Linux
		Lunch menu... pick a combo:
		1. generic-eng
		2. simulator
		3. full_passion-userdebug
		4. full_crespo4g-userdebug
		5. full_crespo-userdebug
		Which would you like? [generic-eng]
	这是从4.2 / Jelly Bean的列表：
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
		Which would you like? [full-eng]
	这些选择不是静态的。最依赖于当前envsetup.sh运行的AOSP中的内容。事实上，它们使用脚本定义的add_lunch_combo（）函数单独添加。例如，在2.3 / Gingerbread中，envsetup.sh
默认添加generic-eng和simulator：
		# add the default one here
		add_lunch_combo generic-eng
		# if we're on linux, add the simulator. There is a special case
		# in lunch to deal with the simulator
		if [ "$(uname)" = "Linux" ] ; then
		add_lunch_combo simulator
		fi
	在4.2 / Jelly Bean中，模拟器不再是有效的目标，而envsetup.sh则改为：
		# add the default one here
		add_lunch_combo full-eng
		add_lunch_combo full_x86-eng
		add_lunch_combo vbox_x86-eng
		add_lunch_combo full_mips-eng
	envsetup.sh还包括可以找到的所有供应商提供的脚本。以下是2.3 / Gingerbread中的完成方法：
		# Execute the contents of any vendorsetup.sh files we can find.
		for f in `/bin/ls vendor/*/vendorsetup.sh vendor/*/build/vendorsetup.sh device/*
		/*/vendorsetup.sh 2> /dev/null`
		do
		echo "including $f"
		. $f
		done
		unset f
	以下是4.2 / Jelly Bean中的完成方法：
		# Execute the contents of any vendorsetup.sh files we can find.
		for f in `/bin/ls vendor/*/vendorsetup.sh vendor/*/*/vendorsetup.sh device/*/*/v
		endorsetup.sh 2> /dev/null`
		do
		echo "including $f"
		. $f
		done
		unset f
	那么这就是你最近看到的菜单。请注意，菜单要求您选择组合。基本上，这是TARGET_PRODUCT和TARGET_BUILD_VARIANT的组合，2.3 / Gingerbread中的模拟器除外。该菜单提供默认
组合，但其他组合仍然有效，可以作为参数在命令行中传递给'lunch'。例如，在2.3 / Gingerbread中，您可以这样做：
		$ lunch generic-user
		============================================
		PLATFORM_VERSION_CODENAME=REL
		PLATFORM_VERSION=2.3.4
		TARGET_PRODUCT=generic
		TARGET_BUILD_VARIANT=user
		TARGET_SIMULATOR=false
		TARGET_BUILD_TYPE=release
		TARGET_BUILD_APPS=
		TARGET_ARCH=arm
		HOST_ARCH=x86
		HOST_OS=linux
		HOST_BUILD_TYPE=release
		BUILD_ID=GINGERBREAD
		============================================
		$ lunch full_crespo-eng
		============================================
		PLATFORM_VERSION_CODENAME=REL
		PLATFORM_VERSION=2.3.4
		TARGET_PRODUCT=full_crespo
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
	一旦‘lunch’完成了一个’generic-eng‘组合的运行，它将在当前shell中设置表4-1中描述的环境变量，为构建系统提供所需的配置信息。
	
	当然，如果你厌倦了总是输入build/envsetup.sh和‘lunch’，你需要做的就是将build/buildspec.mk.default复制到顶级目录中，重命名为buildspec.mk，然后编辑它匹配通过运行这
些命令设置的配置。该文件已包含您需要提供的所有变量;这只是一个取消对相应行的注释和适当设置值的问题。一旦你做完了，你所要做的就是去AOSP的目录并直接调用make。你可以跳
过envsetup.sh和lunch。

##功能定义(Function Definitions)
	由于构建系统相当大(build/core/ 单独存在40多个.mk文件)，因此可以尽可能多地重用代码。这就是为什么构建系统在definitions.mk文件中定义了大量函数的原因。该文件实际上是
构建系统中最大的一个，大约60KB，在2.3 / Gingerbread中大约有1,800行makefile代码上有大约140个函数。它仍然是4.2 / Jelly Bean中构建系统中最大的文件，大小为73KB，170个函
数，以及约2,100行的makefile代码。函数提供各种操作，包括文件查找（例如，allmakefiles-under和all-c-files-under），转换（例如，transform-c-to-o和transform-java-to-classes.jar），
复制 （例如，拷贝文件到目标）和实用程序（例如，我的目录）
	这些功能不仅在构建系统组件的其余部分中使用，作为其核心库，但它们有时也直接用在模块的Android.mk文件中。以下是计算器应用的Android.mk的示例代码片段：
			LOCAL_SRC_FILES := $(call all-java-files-under, src)
	虽然完全描述definitions.mk不在本书的范围之内，你应该很容易自己去探索它。如果没有别的，大部分功能之前都有一个解释他们做什么的评论。以下是2.3 / Gingerbread的一个例子：
			###########################################################
			## Find all of the java files under the named directories.
			## Meant to be used like:
			## SRC_FILES := $(call all-java-files-under,src tests)
			###########################################################
			define all-java-files-under
			$(patsubst ./%,%, \
			$(shell cd $(LOCAL_PATH) ; \
			find $(1) -name "*.java" -and -not -name ".*") \
			)
			endef
			
##主要编译包(Main Make Recipes)
	在这一点上，您可能会想知道哪些好东西实际上是生成的。各种镜像如RAM磁盘生成或SDK如何放在一起，嗯，我希望你不要抱怨，但我一直保持最好的到最后。所以不用多说，看看
在build/core/（不是顶级的）Makefile。该文件以无害的看法开始：
		# Put some miscellaneous rules here
	但不要被愚弄，这是一些最好的肉。下面是生成RAM磁盘的代码段，例如：2.3 / Gingerbread：
		# -----------------------------------------------------------------
		# the ramdisk
		INTERNAL_RAMDISK_FILES := $(filter $(TARGET_ROOT_OUT)/%, \
		$(ALL_PREBUILT) \
		$(ALL_COPIED_HEADERS) \
		$(ALL_GENERATED_SOURCES) \
		$(ALL_DEFAULT_INSTALLED_MODULES))
		BUILT_RAMDISK_TARGET := $(PRODUCT_OUT)/ramdisk.img
		# We just build this directly to the install location.
		INSTALLED_RAMDISK_TARGET := $(BUILT_RAMDISK_TARGET)
		$(INSTALLED_RAMDISK_TARGET): $(MKBOOTFS) $(INTERNAL_RAMDISK_FILES) | $(MINIGZIP)
		$(call pretty,"Target ram disk: $@")
		$(hide) $(MKBOOTFS) $(TARGET_ROOT_OUT) | $(MINIGZIP) > $@
	而这里是创建用于在同一AOSP版本中检查空中（OTA）更新的证书包的代码段：
		# -----------------------------------------------------------------
		# Build a keystore with the authorized keys in it, used to verify the
		# authenticity of downloaded OTA packages.
		#
		# This rule adds to ALL_DEFAULT_INSTALLED_MODULES, so it needs to come
		# before the rules that use that variable to build the image.
		ALL_DEFAULT_INSTALLED_MODULES += $(TARGET_OUT_ETC)/security/otacerts.zip
		$(TARGET_OUT_ETC)/security/otacerts.zip: KEY_CERT_PAIR :=
		$(DEFAULT_KEY_CERT_PAIR)
		$(TARGET_OUT_ETC)/security/otacerts.zip: $(addsuffix .x509.pem,
		$(DEFAULT_KEY_CERT_PAIR))
		$(hide) rm -f $@
		$(hide) mkdir -p $(dir $@)
		$(hide) zip -qj $@ $<
		.PHONY: otacerts
		otacerts: $(TARGET_OUT_ETC)/security/otacerts.zip
	显然，这里有很多可以适用的，但是看看Makefile有关如何创建以下任何内容的信息：
		• Properties (including the target’s /default.prop and /system/build.prop).
		• RAM disk.
		• Boot image (combining the RAM disk and a kernel image).
		• NOTICE files: These are files required by the AOSP’s use of the Apache Software	License (ASL). Have a look at the ASL for more information about NOTICE files.
		• OTA keystore.
		• Recovery image.
		• System image (the target’s /system directory).
		• Data partition image (the target’s /data directory).
		• OTA update package.
		• SDK.
	不过，有些事情不在这个文件中：
	
	Kernel images
		不要寻找建立这些的任何规则。官方AOSP版本没有内核部分 - 附录E中列出的一些第三方项目，然而，实际上将包内核源直接分配到它们分发的AOSP中。相反，您需要为您的目标找
到一个Androidized内核，与AOSP分开构建，并将其提供给AOSP。您可以在device/中的设备中找到一些示例。例如，在2.3/Gingerbread中，device/samsung/crespo/包含一个内核映像
(文件叫做kernel)和一个用于Crespo的WiFi的可加载模块(bcm4329.ko file)。这两个都是在AOSP之外构建的，并以二进制形式复制到树中以便与构建的其余部分一起包含。
	
	NDK
		虽然构建NDK的代码在AOSP中，但它与AOSP的build/中的构建系统完全不同。 相反，NDK的构建系统是在ndk/build/中。我们将讨论如何构建NDK。
		
	CTS
		构建CTS的规则在build/core/tasks/cts.mk中。
		
##Cleaning
	正如我前面提到的，make clean是基本等同于擦除out/目录。’clean‘的目标本身在main.mk中定义。然而，还有其他清理目标。最值得注意的是，在cleanbuild.mk中定义的installclean
会在您更改TARGET_PRODUCT，TARGET_BUILD_VARIANT或PRODUCT_LOCALES时自动调用。所以，如果我最开始为Android2.3构建了generic-eng联合体并在其后使用lunch更改联合体为full-eng，
那在下一次我开始make时，其中的一些构建输出会自动的被installclean修剪：
		$ make -j16
		============================================
		PLATFORM_VERSION_CODENAME=REL
		PLATFORM_VERSION=2.3.4
		TARGET_PRODUCT=full
		TARGET_BUILD_VARIANT=eng
		...
		============================================
		*** Build configuration changed: "generic-eng-{mdpi,nodpi}" -> "full-eng-{en_US,
		en_GB,fr_FR,it_IT,de_DE,es_ES,mdpi,nodpi}"
		*** Forcing "make installclean"...
		*** rm -rf out/target/product/generic/data/* out/target/product/generic/data-qem
		u/* out/target/product/generic/userdata-qemu.img out/host/linux-x86/obj/NOTICE_F
		ILES out/host/linux-x86/sdk out/target/product/generic/*.img out/target/product/
		generic/*.txt out/target/product/generic/*.xlb out/target/product/generic/*.zip
		out/target/product/generic/data out/target/product/generic/obj/APPS out/target/p
		roduct/generic/obj/NOTICE_FILES out/target/product/generic/obj/PACKAGING out/tar
		get/product/generic/recovery out/target/product/generic/root out/target/product/
		generic/system out/target/product/generic/dex_bootjars out/target/product/generi
		c/obj/JAVA_LIBRARIES
		*** Done with the cleaning, now starting the real build.
	与clean相反，installclean不会清除整个的out/目录。相反，只需要重新构建在组合配置更改的那部分。还有一个clobber目标，基本上是和clean一样。
	
##模块构建模板(Module Build Templates)
	我刚刚描述的是构建系统的架构和其核心组件的机制。阅读完毕后，您应该从上而下的角度对Android的构建方式有更好的了解。然而，很少有这一点渗透到AOSP模块的Android.mk文
件的级别。该系统实际上已经被构建，使得模块构建配方与构建系统的内部结构几乎是独立的。相反，提供构建模板，以便模块作者可以适当地构建其模块。每个模板都针对特定类型的
模块量身定制，模块作者可以使用一组记录的变量，所有这些都以LOCAL_为前缀，以调制模板的行为和输出。当然，模板和底层支持文件（主要是base_rules.mk）与构建系统的其余部
分密切相关，以正确处理每个模块的构建输出。但这是模块的作者看不见的。
	这些模板本身与build/core/中的构建系统的其余部分位于相同的位置。Android.mk通过include指令访问它们。以下是一个例子：
	
			include $(BUILD_PACKAGE)
	
	如您所见，Android.mk文件实际上并不包含名称为.mk的模板。相反，它们包括一个设置为相应的.mk文件的变量。表4-2提供了可用模块模板的完整列表。
	以下是以Android2.3中的Service Manager(frameworks/base/cmds/servicemanager/)的Android.mk为例：
	
		LOCAL_PATH:= $(call my-dir)
		include $(CLEAR_VARS)
		LOCAL_SHARED_LIBRARIES := liblog
		LOCAL_SRC_FILES := service_manager.c binder.c
		LOCAL_MODULE := servicemanager
		ifeq ($(BOARD_USE_LVMX),true)
		LOCAL_CFLAGS += -DLVMX
		endif
		include $(BUILD_EXECUTABLE)
		
	以下是Android2.3的桌面时钟(packages/app/DeskClock/)的例子：
	
		LOCAL_PATH:= $(call my-dir)
		include $(CLEAR_VARS)
		LOCAL_MODULE_TAGS := optional
		LOCAL_SRC_FILES := $(call all-java-files-under, src)
		LOCAL_PACKAGE_NAME := DeskClock
		LOCAL_OVERRIDES_PACKAGES := AlarmClock
		LOCAL_SDK_VERSION := current
		include $(BUILD_PACKAGE)
		include $(call all-makefiles-under,$(LOCAL_PATH))
		
	如您所见，两个模块中基本上使用相同的结构，尽管它们提供非常不同的输入，并产生非常不同的输出。还要注意Desk Clock的Android.mk的最后一行，它基本上包括所有子目录的
Android.mk文件。如前所述，构建系统在层次结构中查找第一个makefile，并不会在找到目录的目录下面的任何子目录中查找，因此需要手动调用它们。很显然，这得代码只是出来在
底层去寻找所有的makefile。但是，AOSP的某些部分将根据配置明确列出子目录或有条件地选择它们。

	http://source.android.com上的文档用于提供所有LOCAL_ *变量的详尽列表及其含义和用途。不幸的是，在撰写本文时，此列表已不再可用。然而，build/core/build-system.html
文件包含该列表的早期版本，您应该引用该文件，直到最新列表再次可用。以下是一些最常遇到的LOCAL_ *变量：

		LOCAL_PATH
			通常通过调用$(call my-dir)来提供当前模块源的路径。
		LOCAL_MODULE
			属于此模块的构建输出的名称。实际的文件名或输出及其位置将取决于您包含的构建模板。例如，如果设置为foo，并且您构建一个可执行文件，则最终的可执行文件将是一个名
		为foo的命令，它将被放在目标的/system/bin/目录中。如果LOCAL_MODULE设置为libfoo，并且包括BUILD_SHARED_LIBRARY而不是BUILD_EXECUTABLE，构建系统将生成libfoo.so并将
		其放在/system/lib/中。请注意，您在此处提供的名称对于正在构建的特定模块类（即构建模板类型）必须是唯一的。例如，不能有两个libfoo.so库。预计在将来某个时候，模块名
		称必须是全局唯一的（即跨所有模块类）。
		
		LOCAL_SRC_FILES
			用于构建模块的源文件。您可以通过使用构建系统定义的函数之一来提供这些功能，比如桌面时钟使用all-java-files-under，或者您可以像Service Manager一样显式列出文件。
			
		LOCAL_PACKAGE_NAME
			与所有其他模块不同，app使用此变量而不是LOCAL_MODULE提供其名称，您可以通过比较前面显示的两个Android.mk文件来看到。
			
		LOCAL_SHARED_LIBRARIES
			使用它来列出您的模块所依赖的所有库。如前所述，Service Manager对liblog的依赖使用此变量进行指定。
			
		LOCAL_MODULE_TAGS
			如前所述，这允许您控制此模块构建的TARGET_BUILD_VARIANT。通常，这应该被设置为optional。
			
		LOCAL_MODULE_PATH
			使用它来覆盖您正在构建的模块类型的默认安装位置。
			
	了解更多LOCAL_ *变量的一个好方法是查看AOSP中现有的Android.mk文件。此外，clear_vars.mk包含已清除的变量的完整列表。所以虽然没有给你每个人的意思，但它肯定列出了所
有这些。

	此外，除了影响全局AOSP的清洁目标之外，每个模块都可以通过提供CleanSpec.mk来定义自己的清理规则，就像模块提供的Android.mk文件一样。不像后者，不过前者不是必需的。
默认情况下，构建系统具有每种类型的模块的清除规则。但是，您可以在CleanSpec.mk中指定自己的规则，以防您的模块的构建在构建系统默认情况下不会生成，因此通常不会知道如何
清理。

##输出(output)
	现在我们已经看到了构建系统的工作原理，以及模块使用的构建模板，我们来看看它在out/中创建的输出。在相当高的层次上，构建输出分为三个阶段，两种模式，一种用于主机，一
种用于目标：
		
		1、中间体是使用模块源生成的。这些中间体的格式和位置取决于模块的来源。例如它们可能是基于C/C ++代码的.o文件，也可能是基于Java的代码的.jar文件。
		2、中间体由构建系统用于创建实际的二进制文件和软件包：以.o文件为例，将它们链接到一个实际的二进制文件中。
		3、二进制程序和程序包被组合在一起构建系统所请求的最终输出。例如，二进制文件被复制到包含root和/system文件系统的目录中，并且生成这些文件系统的镜像以便在实际的设
			 备上使用。
			 
	out/主要分为两个目录，反映其操作模式：host/和target/。在每个目录中，您将找到一些包含构建过程中生成的各种中间体的obj/目录。大多数这些存储在前面提到的名为“BUILD_*”
宏的子目录中，并在构建系统的操作过程中提供了特定的补充目的：

		• EXECUTABLES/
		• JAVA_LIBRARIES/
		• SHARED_LIBRARIES/
		• STATIC_LIBRARIES/
		• APPS/
		• DATA/
		• ETC/
		• KEYCHARS/
		• PACKAGING/
		• NOTICE_FILES/
		• include/
		• lib/
	
	您可能最感兴趣的目录是out/target/product/PRODUCT_DEVICE/。这就是相应产品配置的.mk中定义的PRODUCT_DEVICE的输出镜像所在的位置。表4-3说明了该目录的内容。
	请查看第2章，重新了解根文件系统，/system和/data。实际上，当内核引导时，它将挂载RAM磁盘映像并执行内部的/init。这个二进制文件依次运行/init.rc脚本，它会
将/system和/data镜像挂载在各自的位置。我们将在第6章中回到根文件系统布局和引导时的系统操作。
	