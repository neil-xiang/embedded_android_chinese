> 翻译：[neil-xiang](https://github.com/neil-xiang)
> 校对：

###AOSP的基本技巧(Basic AOSP Hacks)
	你买这本书最有可能的原因是：你修改AOSP来适用于你的需求。在接下来的几页中，我们会寻找一些你想要尝试的最明显的技巧。当然，我们只是在这里设置与构建系统相关的部分，
这是您可能希望开始的部分。

##添加一个设备(Adding a Device)
	添加自定义设备很可能是您阅读本书的原因列表中最重要的项目之一（如果不是最高的）。我要告诉你如何做，所以你可能想要标记这个部分。当然，我其实只是向你展示这个作品
的构建方面。将Android移植到新硬件方面还有很多步骤。但是，将新设备添加到构建系统中一定是您首先做的事情之一。幸运的是，这件事比较简单。
	为了本次演习的目的，假设您为名为ACME的公司工作，并且您负责提供其最新的Gizmo：CoyotePad旨在成为玩所有鸟类游戏的最佳平台。让我们开始在device/中为我们的新设备创建
一个条目：

		$ cd ~/android/aosp-2.3.x
		$ . build/envsetup.sh
		$ mkdir -p device/acme/coyotepad
		$ cd device/acme/coyotepad
		
	我们在这里需要的第一件事就是创建一个AndroidProducts.mk文件来描述可以为CoyotePad构建的各种AOSP产品：
	
			PRODUCT_MAKEFILES := $(LOCAL_DIR)/full_coyotepad.mk
			
	虽然我们可以描述几个产品（参见build/target/product/AndroidProd ucts.mk作为示例），典型的情况是指定一个，在这种情况下，它在full_coyotepad.mk中描述：
	
			$(call inherit-product, $(SRC_TARGET_DIR)/product/languages_full.mk)
			# If you're using 4.2/Jelly Bean, use full_base.mk instead of full.mk
			$(call inherit-product, $(SRC_TARGET_DIR)/product/full.mk)
			DEVICE_PACKAGE_OVERLAYS :=
			PRODUCT_PACKAGES +=
			PRODUCT_COPY_FILES +=
			PRODUCT_NAME := full_coyotepad
			PRODUCT_DEVICE := coyotepad
			PRODUCT_MODEL := Full Android on CoyotePad, meep-meep
			
	值得仔细看看这个makefile。首先，我们使用‘inherit-product’功能来告诉构建系统将其他产品描述作为我们的基础。这使我们能够建立在其他人的工作上，而不必从头开始指定我
们想要包括的AOSP。languages_full.mk将拉入大量的区域设置，并且full.mk将确保我们获得与使用full-eng组合一样构建的同一组模块。
	关于其他变量：
	
		DEVICE_PACKAGE_OVERLAYS
			允许我们指定一个目录，该目录将构成将应用到AOSP源的重叠的基础，从而允许我们用设备特定资源替换默认的包资源。例如，如果您想为Launcher2或其他应用程序设置自定义
			布局或颜色，则会发现此功能非常有用。我们将在下一节中介绍如何使用它。
			
		PRODUCT_PACKAGES
			允许我们指定要包含此产品的软件包，以及我们已经继承的产品中指定的软件包。例如，如果您有定制化app，二进制文件或位于device/acme/coyotepad/中的库，您需要将它们
			添加到这里，以便将它们包含在生成的最终镜像中。注意使用+ =符号。它允许我们附加到变量中的现有值，而不是替换其内容。
			
		PRODUCT_COPY_FILES
			允许我们列出我们想要看到的复制到目标文件系统的特定文件以及需要复制到的位置。每对的目的/源被冒号‘:’分割，每队之前用空格'space'分开。这对于配置文件和预构建的
			二进制文件（如固件映像或内核模块）非常有用。
			
		PRODUCT_NAME
			TARGET_PRODUCT，您可以通过选择lunch组合或将其作为组合参数的一部分传递给lunch，如下所示：
					
					$ lunch full_coyotepad_eng
					
		PRODUCT_DEVICE
			给到客户的实际成品的名称。TARGET_DEVICE来自此变量。PRODUCT_DEVICE必须匹配device/acme/中的一个条目，因为这是构建查找相应的BoardConfig.mk的位置。在这种情况下，
			该变量与我们已经存在的目录的名称相同。
		
		PRODUCT_MODEL
			设置中“关于手机”部分“型号”中提供的本产品的名称。该变量实际上被存储为设备上可访问的ro.prod uct.model全局属性。
			
	版本4.2/Jelly Bean还包括通常设置为Android的PRODUCT_BRAND。然后，该变量的值可用作ro.product.brand全局属性。后者由堆栈的某些部分用于基于设备的供应商的动作。
	现在我们已经描述了该产品，我们还必须通过BoardConfig.mk文件提供有关设备正在使用的板卡的一些信息：
	
			TARGET_NO_KERNEL := true
			TARGET_NO_BOOTLOADER := true
			TARGET_CPU_ABI := armeabi
			BOARD_USES_GENERIC_AUDIO := true
			USE_CAMERA_STUB := true
			
	这是一个很瘦小的BoardConfig.mk，并确保我们实际构建成功。对于该文件的现实版本，可以查看Android2.3的device/samsung/crespo/BoardConfigCommon.mk或者Android4.2
的device/asus/grouper/BoardConfigCommon.mk。
	您还需要提供一个常规的Android.mk来构建您可能已经包含在此设备目录中的所有模块：
		
			LOCAL_PATH := $(call my-dir)
			include $(CLEAR_VARS)
			ifneq ($(filter coyotepad,$(TARGET_DEVICE)),)
			include $(call all-makefiles-under,$(LOCAL_PATH))
			endif
			
	它实际上是首选的操作方式，将所有设备特定的app，二进制文件和库放在设备的目录中，而不是全局的AOSP的其余部分。如果您在这里添加模块，不要忘了还将它们添加到我们前面
描述过的PRODUCT_PACKAGES中。如果您只是将它们放在这里并提供有效的Android.mk文件，那么它们将被构建，但它们不会在最终的镜像中。
	如果您有多个产品共享相同的软件包，您可能需要创建一个包含共享软件包的device/acme/common/目录。你可以在Android4.2的device/generic/目录中看到一个例子。在同一版本中，
您还可以查看device/samsung/maguro/device.mk如何从device/samsung/tuna/device.mk继承一个设备基于另一个设备的示例。
	最后，让我们将envsetup.sh中添加的设备添加到lunch中来关闭该循环。为此，您需要在设备的目录中添加vendorsetup.sh：
		
			add_lunch_combo full_coyotepad-eng
			
	您还需要确保它是可执行的，如果它是可操作的：
	
			$ chmod 755 vendorsetup.sh
			
	现在我们可以回到AOSP的根目录并将我们最新的ACME CoyotePad来展开追逐：
		
			$ croot
			$ . build/envsetup.sh
			$ lunch
			You're building on Linux
			Lunch menu... pick a combo:
			1. generic-eng
			2. simulator
			3. full_coyotepad-eng
			4. full_passion-userdebug
			5. full_crespo4g-userdebug
			6. full_crespo-userdebug
			
			Which would you like? [generic-eng] 3
			============================================
			PLATFORM_VERSION_CODENAME=REL
			PLATFORM_VERSION=2.3.4
			TARGET_PRODUCT=full_coyotepad
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
			$ make -j16
			
	正如你看到的，AOSP以及国内可以识别我们的新设备并打印出了正确的信息。构建完成后，我们还将提供与其他任何AOSP构建相同类型的输出，除了它将是产品特定的目录：
			
			$ ls -al out/target/product/coyotepad/
			total 89356
			drwxr-xr-x 7 karim karim 4096 2011-09-21 19:20 .
			drwxr-xr-x 4 karim karim 4096 2011-09-21 19:08 ..
			-rw-r--r-- 1 karim karim 7 2011-09-21 19:10 android-info.txt
			-rw-r--r-- 1 karim karim 4021 2011-09-21 19:41 clean_steps.mk
			drwxr-xr-x 3 karim karim 4096 2011-09-21 19:11 data
			-rw-r--r-- 1 karim karim 20366 2011-09-21 19:20 installed-files.txt
			drwxr-xr-x 14 karim karim 4096 2011-09-21 19:20 obj
			-rw-r--r-- 1 karim karim 327 2011-09-21 19:41 previous_build_config.mk
			-rw-r--r-- 1 karim karim 2649750 2011-09-21 19:43 ramdisk.img
			drwxr-xr-x 11 karim karim 4096 2011-09-21 19:43 root
			drwxr-xr-x 5 karim karim 4096 2011-09-21 19:19 symbols
			drwxr-xr-x 12 karim karim 4096 2011-09-21 19:19 system
			-rw------- 1 karim karim 87280512 2011-09-21 19:20 system.img
			-rw------- 1 karim karim 1505856 2011-09-21 19:14 userdata.img
			
	另外，请看system/中的build.prop文件。它包含可在运行时在目标上提供的各种全局属性，并且与我们的配置和构建相关：
	
			# begin build properties
			# autogenerated by buildinfo.sh
			ro.build.id=GINGERBREAD
			ro.build.display.id=full_coyotepad-eng 2.3.4 GINGERBREAD eng.karim.20110921.1908
			49 test-keys
			ro.build.version.incremental=eng.karim.20110921.190849
			ro.build.version.sdk=10
			ro.build.version.codename=REL
			ro.build.version.release=2.3.4
			ro.build.date=Wed Sep 21 19:10:04 EDT 2011
			ro.build.date.utc=1316646604
			ro.build.type=eng
			ro.build.user=karim
			ro.build.host=w520
			ro.build.tags=test-keys
			ro.product.model=Full Android on CoyotePad, meep-meep
			ro.product.brand=generic
			ro.product.name=full_coyotepad
			ro.product.device=coyotepad
			ro.product.board=
			ro.product.cpu.abi=armeabi
			ro.product.manufacturer=unknown
			ro.product.locale.language=en
			ro.product.locale.region=US
			ro.wifi.channels=
			ro.board.platform=
			# ro.build.product is obsolete; use ro.product.device
			ro.build.product=coyotepad
			# Do not try to parse ro.build.description or .fingerprint
			ro.build.description=full_coyotepad-eng 2.3.4 GINGERBREAD eng.karim.20110921.190
			849 test-keys
			ro.build.fingerprint=generic/full_coyotepad/coyotepad:2.3.4/GINGERBREAD/eng.kari
			m.20110921.190849:eng/test-keys
			# end build properties
			...
				
	你可以想象，这里还有很多事情要做，以确保AOSP在我们的硬件上运行。但是前面的步骤给了我们起点。然而，通过隔离单个目录中的单板特定更改，此配置将简化将CoyotePad的附
加支持添加到下一个版本的AOSP中。实际上，它只是将相应的目录复制到新的AOSP的device/目录，并调整其中的代码以使用新的API。

##添加一个app(Adding an App)
	添加一个应用程序到你的板子上是比较直接的。试着将用Eclipse和默认的SDK创建一个‘HelloWorld!’app作为起点。Eclipse中的所有新的Android项目都默认是'Hello World!'.然后
将该应用程序从Eclipse工作区复制到其目标：

		$ cp -a ~/workspace/HelloWorld ~/android/aosp-2.3.x/device/acme/coyotepad/
		
	然后，您必须在aosp-root/device/acme/coyotepad/HelloWorld/中创建一个Android.mk文件来构建该应用程序：
			
			LOCAL_PATH:= $(call my-dir)
			include $(CLEAR_VARS)
			LOCAL_MODULE_TAGS := optional
			LOCAL_SRC_FILES := $(call all-java-files-under, src)
			LOCAL_PACKAGE_NAME := HelloWorld
			include $(BUILD_PACKAGE)
			
	鉴于我们将此模块标记为可选项，默认情况下不会包含在AOSP构建中。要包括它，您需要将其添加到CoyotePad的full_coyotepad.mk中列出的PRODUCT_PACKAGES。
	如果，而不是仅添加您的应用程序到你的板子上，您想在全球范围内为AOSP生成的所有产品以及现有常用应用程序添加默认应用程序，您需要将其放在packages/app中，而不是您的板
子目录下。您还需要修改内置的.mk文件，如aosp-root/build/target/product/core.mk，以便默认构建您的应用程序。不过建议不要这样做，因为它不是很便携，因为它需要你对每个新
的AOSP版本进行修改。如前所述，最好尽可能多地在device/acme/coyotepad中进行自定义修改。

##添加一个app叠加(Adding an App Overlay)
	有时您实际上并不想添加一个应用程序，而是修改AOSP中默认的现有应用程序。这就是应用程序的重叠。覆盖是AOSP中包含的一种机制，允许设备制造商更改所提供的资源(例如app)，
而无需实际修改AOSP中包含的原始资源.要使用此功能，您必须创建一个覆盖树，并告知构建系统。重叠式最简单位置在特定于设备的目录中，例如我们在上一节中创建的目录：

			$ cd device/acme/coyotepad/
			$ mkdir overlay
			
	要告诉构建系统考虑到这个覆盖，我们需要修改我们的full_coyotepad.mk：
	
			DEVICE_PACKAGE_OVERLAYS := device/acme/coyotepad/overlay
			
	在这一点上，尽管如此，我们的覆盖没有多少。假设我们要修改一些Launcher2的默认字符串。 我们可以这样做：
	
			$ mkdir -p overlay/packages/apps/Launcher2/res/values
			$ cp aosp-root/packages/apps/Launcher2/res/values/strings.xml overlay/packages/apps/Launcher2/res/values/
			
	然后，您可以修剪本地strings.xml以仅覆盖所需的字符串。最重要的是，您的设备将有一个具有自定义字符串的Launcher2，但默认的Launcher2仍将具有原始字符串。因此，如果有
人依赖于您用于构建另一个产品的相同的AOSP源，那么它们仍然会获得原始字符串。当然，您可以替换大多数资源，包括图像和XML文件。只要将文件放置在与AOSP中相同的层次结构中，
而不是device/acme/coyotepad/overlay/中，它们将被构建系统考虑在内。


##添加一个Native的工具或者后台进程(Adding a Native Tool or Daemon)
	像上面添加一个应用程序的例子一样，您可以添加您的自定义本地工具和后台在device/acme/coyotepad/的子目录。显然，您需要在包含代码的目录中提供一个Android.mk来构建该模块：
			
			LOCAL_PATH:= $(call my-dir)
			include $(CLEAR_VARS)
			LOCAL_MODULE := hello-world
			LOCAL_MODULE_TAGS := optional
			LOCAL_SRC_FILES := hello-world.cpp
			LOCAL_SHARED_LIBRARIES := liblog
			include $(BUILD_EXECUTABLE)
			
	在应用程序的情况下，您还需要确保hello-world是CoyotePad的PRODUCT_PACKAGES的一部分。
	如果您打算将全局的二进制文件全部添加到所有产品构建中，而不是仅在本地添加到您的主板上，则需要知道树中有多个本地工具和后台进程所在的位置。这些里是最重要的：
		
		system/core/ 和 system/
			定制Android二进制文件，意在在Android Framework外部使用或独立的部分。
			
		frameworks/base/cmds/
			与Android Framework紧密耦合的二进制文件。例如，这是Service Manager和installd的位置。
			
		external/
			由导入到AOSP的外部项目生成的二进制文件。例如，strace在这里。
			
	从上面列出的代码生成二进制代码的位置，您还需要将其添加为全局.mk文件之一，如aosp-root/build/target/product/core.mk。然而，如上所述，不建议使用全局添加，因为它们不能轻易转移到较新的AOSP版本。
	
##添加一个本地库(Adding a Native Library)
	像app和二进制文件一样，您还可以为您的主板添加本机库。假设，如上所述，构建库的源位于device/acme/coyotepad/的子目录中，您将需要一个Android.mk来构建库：
	
			LOCAL_PATH:= $(call my-dir)
			include $(CLEAR_VARS)
			LOCAL_MODULE := libmylib
			LOCAL_MODULE_TAGS := optional
			LOCAL_PRELINK_MODULE := false
			LOCAL_SRC_FILES := $(call all-c-files-under,.)
			include $(BUILD_SHARED_LIBRARY)
			
	要使用此库，您必须将其添加到由Android.mk文件列出的任何二进制文件所依赖的库中：
	
			LOCAL_SHARED_LIBRARIES := libmylib
			
	您还可能需要将相关标头添加到与放置库相同位置的include/目录中，以便需要链接到库的代码可以找到这些标题，例如device/acme/coyotepad/include/。
	如果您想将库全局应用于所有AOSP构建，而不仅仅是您的设备，那么您需要更多关于树通常在库中找到的各种位置的信息。首先，您应该知道，与二进制文件不同，在单个模块中使用
了很多库，但是不会适用其他地方的库。因此，这些库通常将被放置在该模块的代码中，而不是位于系统范围内使用库的通常位置。后者通常在以下位置：

		system/core/
			系统许多部分使用的库，包括Android Framework之外的一些。这就是liblog所在的地方。
			
		frameworks/base/libs/
			库文件与框架密切相关。 这是libbinder的地方。
			
		frameworks/native/libs/
			在Android4.2中，许多在Android2.3中frameworks/base/libs/下的库已被移出并进入frameworks/native/libs/。
			
		external/
			由外部项目生成的库导入AOSP。 OpenSSL libssl在这里。
			
	类似地，代替使用CoyotePad特定的包含目录，您可以使用全局目录，如system/core/include/ or frameworks/base/include/ 或者4.2版中frameworks/base/include/.同样，如前所
述，您应该仔细检查是否真正需要这样的全球添加，因为当您尝试将设备移植到下一个版本的Android时，它们将是额外的工作量。