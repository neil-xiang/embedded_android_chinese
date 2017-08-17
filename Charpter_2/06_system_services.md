> 翻译：[Neil-xiang](https://github.com/Neil-xiang)
> 校对：

###系统服务(System Services)
	系统服务是Android的幕后人物.即使Google的应用开发文档中没有明确提及，Android中远程有趣的任何操作都可以通过大约50到70个系统服务之一进行。这些服务合作共同提供了基本上相当于构建在Linux
上的面向对象的操作系统，这正是Binder所建立的所有系统服务的机制。我们刚刚涵盖的本地用户空间实际上被设计为Android系统服务的支持环境。因此，了解什么系统服务存在以及以及它们如何与系统的其
他部分进行交互至关重要。我们已经将其中的一些在Android硬件支持的部分讨论过了。
	图2-4更详细地说明了之前在图2-1中介绍的系统服务。你可以看到，实际上涉及到几个主要的进程。最显著的是系统服务，其主要由Java编码的服务组成，其中有两个以C/C ++编写的服务，其组件都在
system_server进程下运行。系统服务还包含一些通过JNI访问的本地代码允许java代码与Android的底层进行交互。另一个系统服务包含在Media服务中，以mediaserver形式运行。这些服务都以C/C ++编写，并
与媒体相关的组件（如StageFright多媒体框架和音频效果）一起打包。最后，电话应用程序将电话服务与其他应用程序分开放置。注意自从Android4.0/Ice-Cream Sandwich开始，Surface Flinger已经被分离
成独立进程。
		**这里的术语不是我选择的，但是很不幸很有迷惑性。“System Server”进程在同一进程中容纳多个系统服务。‘Media Service’也是一样。“System Server”和“Media Service”都是单数形式，无论其组成的
			系统服务数量如何.当本书涉及“System Servers”时，它是指系统中可用的所有系统服务，而不考虑其运行的进程。所以简而言之，“SystemServer”和“Media Service”都不是“System Servers”的一部分。
			相反，它们是用于运行后者的进程。
			
		*图2-4 系统服务
	请注意，尽管只有少数进程可以容纳整个Android的系统服务，他们似乎独立地通过Binder连接到他们的服务。这是Android2.3/Gingerbread仿真器上的服务实用程序的输出：
	# service list
	Found 50 services:
	0 phone: [com.android.internal.telephony.ITelephony]
	1 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
	2 simphonebook: [com.android.internal.telephony.IIccPhoneBook]
	3 isms: [com.android.internal.telephony.ISms]
	4 diskstats: []
	5 appwidget: [com.android.internal.appwidget.IAppWidgetService]
	6 backup: [android.app.backup.IBackupManager]
	7 uimode: [android.app.IUiModeManager]
	8 usb: [android.hardware.usb.IUsbManager]
	9 audio: [android.media.IAudioService]
	10 wallpaper: [android.app.IWallpaperManager]
	11 dropbox: [com.android.internal.os.IDropBoxManagerService]
	12 search: [android.app.ISearchManager]
	13 location: [android.location.ILocationManager]
	14 devicestoragemonitor: []
	15 notification: [android.app.INotificationManager]
	16 mount: [IMountService]
	17 accessibility: [android.view.accessibility.IAccessibilityManager]
	18 throttle: [android.net.IThrottleManager]
	19 connectivity: [android.net.IConnectivityManager]
	20 wifi: [android.net.wifi.IWifiManager]
	21 network_management: [android.os.INetworkManagementService]
	22 netstat: [android.os.INetStatService]
	23 input_method: [com.android.internal.view.IInputMethodManager]
	24 clipboard: [android.text.IClipboard]
	25 statusbar: [com.android.internal.statusbar.IStatusBarService]
	26 device_policy: [android.app.admin.IDevicePolicyManager]
	27 window: [android.view.IWindowManager]
	28 alarm: [android.app.IAlarmManager]
	29 vibrator: [android.os.IVibratorService]
	30 hardware: [android.os.IHardwareService]
	31 battery: []
	32 content: [android.content.IContentService]
	33 account: [android.accounts.IAccountManager]
	34 permission: [android.os.IPermissionController]
	35 cpuinfo: []
	36 meminfo: []
	37 activity: [android.app.IActivityManager]
	38 package: [android.content.pm.IPackageManager]
	39 telephony.registry: [com.android.internal.telephony.ITelephonyRegistry]
	40 usagestats: [com.android.internal.app.IUsageStats]
	41 batteryinfo: [com.android.internal.app.IBatteryStats]
	42 power: [android.os.IPowerManager]
	43 entropy: []
	44 sensorservice: [android.gui.SensorServer]
	45 SurfaceFlinger: [android.ui.ISurfaceComposer]
	46 media.audio_policy: [android.media.IAudioPolicyService]
	47 media.camera: [android.hardware.ICameraService]
	48 media.player: [android.media.IMediaPlayerService]
	49 media.audio_flinger: [android.media.IAudioFlinger]
	
	以下是Android 4.2/Jelly Bean仿真器的输出：
	root@android:/ # service list
	Found 68 services:
	0 phone: [com.android.internal.telephony.ITelephony]
	1 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
	2 simphonebook: [com.android.internal.telephony.IIccPhoneBook]
	3 isms: [com.android.internal.telephony.ISms]
	4 dreams: [android.service.dreams.IDreamManager]
	5 commontime_management: []
	6 samplingprofiler: []
	7 diskstats: []
	8 appwidget: [com.android.internal.appwidget.IAppWidgetService]
	9 backup: [android.app.backup.IBackupManager]
	10 uimode: [android.app.IUiModeManager]
	11 serial: [android.hardware.ISerialManager]
	12 usb: [android.hardware.usb.IUsbManager]
	13 audio: [android.media.IAudioService]
	14 wallpaper: [android.app.IWallpaperManager]
	15 dropbox: [com.android.internal.os.IDropBoxManagerService]
	16 search: [android.app.ISearchManager]
	17 country_detector: [android.location.ICountryDetector]
	18 location: [android.location.ILocationManager]
	19 devicestoragemonitor: []
	20 notification: [android.app.INotificationManager]
	21 updatelock: [android.os.IUpdateLock]
	22 throttle: [android.net.IThrottleManager]
	23 servicediscovery: [android.net.nsd.INsdManager]
	24 connectivity: [android.net.IConnectivityManager]
	25 wifi: [android.net.wifi.IWifiManager]
	26 wifip2p: [android.net.wifi.p2p.IWifiP2pManager]
	27 netpolicy: [android.net.INetworkPolicyManager]
	28 netstats: [android.net.INetworkStatsService]
	29 textservices: [com.android.internal.textservice.ITextServicesManager]
	30 network_management: [android.os.INetworkManagementService]
	31 clipboard: [android.content.IClipboard]
	32 statusbar: [com.android.internal.statusbar.IStatusBarService]
	33 device_policy: [android.app.admin.IDevicePolicyManager]
	34 lock_settings: [com.android.internal.widget.ILockSettings]
	35 mount: [IMountService]
	36 accessibility: [android.view.accessibility.IAccessibilityManager]
	37 input_method: [com.android.internal.view.IInputMethodManager]
	38 input: [android.hardware.input.IInputManager]
	39 window: [android.view.IWindowManager]
	40 alarm: [android.app.IAlarmManager]
	41 vibrator: [android.os.IVibratorService]
	42 battery: []
	43 hardware: [android.os.IHardwareService]
	44 content: [android.content.IContentService]
	45 account: [android.accounts.IAccountManager]
	46 user: [android.os.IUserManager]
	47 permission: [android.os.IPermissionController]
	48 cpuinfo: []
	49 dbinfo: []
	50 gfxinfo: []
	51 meminfo: []
	52 activity: [android.app.IActivityManager]
	53 package: [android.content.pm.IPackageManager]
	54 scheduling_policy: [android.os.ISchedulingPolicyService]
	55 telephony.registry: [com.android.internal.telephony.ITelephonyRegistry]
	56 display: [android.hardware.display.IDisplayManager]
	57 usagestats: [com.android.internal.app.IUsageStats]
	58 batteryinfo: [com.android.internal.app.IBatteryStats]
	59 power: [android.os.IPowerManager]
	60 entropy: []
	61 sensorservice: [android.gui.SensorServer]
	62 media.audio_policy: [android.media.IAudioPolicyService]
	63 media.camera: [android.hardware.ICameraService]
	64 media.player: [android.media.IMediaPlayerService]
	65 media.audio_flinger: [android.media.IAudioFlinger]
	66 drm.drmManager: [drm.IDrmManagerService]
	67 SurfaceFlinger: [android.ui.ISurfaceComposer]
	
	不幸的是，这些服务如何工作的资料都不是很多。您必须查看每个服务的源代码，以准确了解其工作原理以及如何与其他服务进行交互。
	
	##服务管理器及Binder交互(Service Manager and Binder Interaction)
	
	如我之前解释的那样，binder机制作为面向对象的RPC/IPC系统服务的基础。系统中的进程通过Binder来调用系统服务，然而它首先必须要有一个句柄。例如，Binder允许app开发人员通过WakeLock基类的acquire()
方法来通过电源管理调用一个唤醒锁请求。但是在那个请求之前，开发人员必须首先得到电源管理服务的一个句柄函数。就如我们在接下来我们看到的那样，app开发API实际上隐藏了它如何得到对开发人员很抽象的句
柄函数详细的细节，但是在这之下，系统服务的句柄函数都是通过服务管理器来查找的，如图2-5所示。
		*图2-5 服务管理器及Binder交互
	可以将服务管理器看着是一个系统中可以提供的服务的一个黄页。如果一个系统服务没有在服务管理器中注册，那它对于系统的其它部分是不可见的。为了提供索引的功能，服务管理器在任何服务之前由‘init’启动。
然后打开/dev/binder并使用一个特殊的调用ioctl()来讲它自己设置为Binder的上下文管理器(Binder's Context Manager)(图2-5中的A1)。此后，很多系统进程试图与ID为0的Binder(也就是在很多代码中被称为‘魔法’
('magic')Binder或者‘魔法对象’(‘magic object’))通信，实际上是通过Binder和服务管理器通信。
	当系统服务启动时，它会将每个单独的服务注册并通过服务管理器实例化(图2-5种的A2)。以后当app是如与系统服务通信是，比如说电源管理服务，，它首先向服务管理器请求服务的句柄函数(B1)并调用那个服务的
方法(B2).相反，对app中运行的服务组件调用直接通过Binder(C1)而不是通过服务管理器来查找。
	服务管理器也被一些命令行工具以特殊的方式使用，比如'dumpsys'工具，它允许你将一个信号的状态或者所有的系统服务转存下来。为了得到所有服务的列表，‘dumpsys’循环的得到每个系统服务(D1),没一次交互都
得到第n各服务直到全部得到。为了得到每一个服务，‘dumpsys’为让服务管理器定位到那个具体的(D2).当得到服务的句柄函数后，'dumpsys'会调用系统的dump()函数来转存服务的状态(D3)并在终端上显示出来.

	##服务请求(Calling on Services)
	正如我之前说的，刚刚我解释的东西对于普通的app开发人员是不可见的，例如，这里有一段代码段，这允许我们使用常规app开发API在应用程序中捕获唤醒锁：
	
	PowerManager pm = (PowerManager) getSystemService(POWER_SERVICE);
	PowerManager.WakeLock wakeLock = pm.newWakeLock(PowerManager.FULL_WAKE_LOCK, "myPreciousWakeLock");
	wakeLock.acquire(100);
	
	注意在这里我们没有看到任何服务管理器的迹象。相反的，我们使用了getSystemService()并传递了POWER_SERVICE参数给它。然而，在内部，getSystemService()代码确实使用了服务管理器来定位电源管理服务，所以
我们能成功创建一个唤醒锁。附录B显示如何添加系统服务并通过getSystemService()使其可用。

	##一个服务例子:活动管理器(A Service Example: the Activity Manager)
	
	虽然本书不能包含每一个系统服务，但是让我们来快速的浏览一下活动管理器(Activity Manager)，核心系统服务中的一个。在Android2.3/Gingerbread,Activity Manager源代码实际上包含30多个文件和20,000行代码。
如果在Android内部有一个内核的话，这个服务就非常接近它。它负责开始一个新的组件，比如Activities和Services，along with the fetching of Content Providers and intent broadcasting。如果你碰到过可怕的
ANR(Application Not Responding)对话框的话，它的背后就有Activity Manager。它还涉及内核低内存处理时OOM调整的维护、权限管理、任务管理器等。
	例如，当用户在他们的桌面上点击一个图标的话，最先发生的事情就是Launcher的onClick()的回调被调用(Launcher是在AOSP的内的一个默认的app程序包，负责处理和用户及主屏的主界面)。为了处理这个事件，Launcher
会通过Binder调用Activity Manager服务的startActivity()方法。这时服务会调用startViaZygote()方法，这个方法会打开一个端口给Zygote并让它开始一个Activity。在阅读本章的最后一节之后，这一切可能会更容易理解。
	如果你对Linux的内部熟悉的话，一个很好的思路是，可以看着Activity Manager对于Android就像内核源码中kernel/目录下的内容对于Liunx一样。这很重要。