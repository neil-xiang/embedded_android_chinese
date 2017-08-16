> 翻译：[neil-xiang](https://github.com/neil-xiang)

> 校对：

### 运行Android(Running Android)
	当编译完成后，你所需要做的就是开始模拟器并运行你定制化的镜像：
		$  emulator &
	这个命令会开始模拟器窗口并引导完整的Android环境。
	现在你可以像运行在真正的设备上一样和AOSP交互了。虽然你的显示器并不一定是触摸屏，然而，你可以使用你的鼠标就像使用你的手指一样。单击就像点击一样。按住鼠标按键来拖动，
四处移动，松开鼠标按键就意味着你的手指离开了触摸屏。你还可以在你的装置上拥有全键盘，能找到在QWERT键盘手机上能找到的所有按键，尽管你可以使用常规键盘在文本框中输入文本。

	尽管它的特点和现实主义，仿真器确实有它的问题。一方面，需要一些时间才能启动。首次启动将需要最长时间，因为Dalvik正在为手机上运行的应用程序创建JIT缓存。请注意，Dalvik
缓存的创建不是模拟器唯一的。无论你运行Android的是什么类型的设备，现代的Dalvik都需要一个JIT缓存，无论是在启动时创建，还是在第7章中讲到的在构建时创建。
	即使在第一次启动后，你可能会发现模拟器很繁重，尤其是在‘修改-编译-测试’循环中。而且，它并不完美地模仿一切。例如，当使用F11或F12进行旋转时，它通常很难打开旋转变化事件。
但是，这对于应用程序开发人员来说大多是一个问题。
	如果由于任何原因关闭了你配置，构建和启动Android的shell，或者如果你需要启动一个新的，并且可以访问从构建创建的所有工具和二进制文件，您必须再次调用envsetup.sh脚本和‘lunch’
命令，以便设置环境变量。以下是新shell的命令，例如：
	$ cd ~/android/aosp-2.3.x
	$ emulator &
	No command 'emulator' found, did you mean:
	Command 'qemulator' from package 'qemulator' (universe)
	emulator: command not found
	$ . build/envsetup.sh
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
	...
	============================================
	$ emulator &
	$
	
	请注意，我们第二次发布仿真器时，shell并没有抱怨该命令已经丢失了。 很多其他Android工具也是如此，比如我们将要看的adb命令。还要注意，我们不需要发出任何make命令，因为我们
已经构建了Android。在这种情况下，我们只需要确保环境变量被正确设置，以便先前版本的结果再次可用。