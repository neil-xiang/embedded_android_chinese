> 翻译：[koffuxu](https://github.com/koffuxu)
> 校对：

# 获取 AOSP

	在前面的章节我已经提到过，官方AOSP可以通过这个网站‘http://android.git.kernel.org‘获取，这是一个支持git接口的网站。当你浏览这个网站的时候你可以看到大量的你可以pull下来的git代码仓库，
可想而知，手动把每个git仓库下载下来是非常繁琐的事情；AOSP包含了数以百计的仓库。但事实上，下载所有的仓库是没有用的，因为在一个项目中只有某几个仓库是需要的。正确的下载AOSP方式是通过
`repo`工具，它可以在同一个位置获得。首先，你先得把`repo`这个工具安装好：

	$ sudo apt-get install curl
	$ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
	$ chmod a+x ~/bin/repo

		**在Ubuntu下，‘~/bin’这个路径如果已经存在，当你登陆的时候会自动添加到你系统的环境变量中。所以，如果你的根目录没有’bin/‘这个目录的话，创建一个，登出系统然后登入，确保这个路径在系
			统的环境变量中。不然的话，就算你安装好`repo` shell也找不到`repo`。
			
		**如果这在Ubntu或你使用的其它版本中无法达到要求，手动添加”PATH=$PATH:~/bin“到你的’~/.profile‘目录下，登出系统然后登入。

		**你不一定要把`repo`放到 ’~/bin‘，但这个一定得在你的目录里。所以，不管你把它放在哪里，只要确保你在命令行操作下能够进入的地方即可。

	尽管`repo` 看起来就是一个简单的shell命令，但它却是一个相当复杂的工具。它可以同时从多个git存储库中提取，以创建一个Android发行版。提取文件的仓库是通过’manifest‘文件提供的，’manifest‘
文件是XML类型的，它描述了那些是需要提取的项目以及它们的位置。事实上’repo‘在’git的上层‘，每个提取的项目是一个单独的git仓库。你可以在在Android的第一个公开版本发行后不久就发表的博客
”Gerrit and Repo, the Android Source Management Tools“中找到是什么原因促使google推出了’repo‘的更多地信息。

		**也许会混淆repo的“Manifest”文件和在APP开发中用于描述APP信息的的“Manifest”（AndroidManifest.xml），其实它们几乎是没有任何关系。它们的格式和使用完全不同。幸运的是，它们几乎不会在同
		一个场景下使用，所以你就不需要太担心这个接下来的解释。

	在使用repo之前，你需要确认git已经在你的系统中安装了，因为默认情况下可能没有：
		$ sudo apt-get install git
		
	现在我们已经安装好repo和git，接下来我们来获取我们自己的AOSP备份

	$ mkdir -p ~/android/aosp-2.3.x
	$ cd ~/android/aosp-2.3.x
	$ repo init -u https://android.googlesource.com/platform/manifest.git -b gingerbread
	$ repo sync


	最后一个命令就需要一些时间来执行，它将按照manifest文件的描述来下载各个项目的代码仓库。总的来说，一份未编译的AOSP源码大概有4G。因此，请记住你的网络带宽，时延性在决定你需要花多少时间
扮演很重要的角色。记住我们下载一个指定的代码分支**Gingerbread**。即第三条命令中的`-b  gingerbread`参数。如果你忽略这这部分，你下载下来的代码是`master`分支的。这个分支是有很多人的经验
之作，也许不能正常的编译和运行，因为这里面包含了开放开发的分支。标记分支，另一方面，许多东西直接拿来使用。如果你计划为AOSP做出贡献，记住Google只接受对主分支的修改。
	
	你可以使用repo的在线帮助来获得repo功能的详细信息:$repo help
	你也可以使用 ‘$ repo help XXXX’ 来获得某个命令的详细信息，比如'$ repo help init'
	例如当你在看'repo sync’的在线帮助时，有一个标记你可能会想要更细致的了解‘-j’，因为它允许并行同步几个git树。如果您有一个慷慨的公司网络连接，并希望加速您的AOSP的下载，这是非常有用的，
repo默认使用4个并行下载。
	‘$ repo sync -j8’
	要获取其它的分支或者是标签也很容易，以下是获取Android4.2/Jelly Bean的示例：
	
	$ mkdir -p ~/android/aosp-4.2
	$ cd ~/android/aosp-4.2
	$ repo init -u https://android.googlesource.com/platform/manifest -b android-4.2_r1
	$ repo sync

	与之前的命令相比，我使用的是特定的版本号而不是版本名称。代号，标签和编号提供了官方标签和版本号的完整列表。你可以通过如下的操作得到所有可以得到的分支和标签：
	$ mkdir ~/android/aosp-branches-tags
	$ cd ~/android/aosp-branches-tags
	$ git clone https://android.googlesource.com/platform/manifest.git
	$ cd manifest
	$ git tag /$ git branch -a
	以上命令后列出的所有内容当然仅限于官方AOSP。有关可能与您的工作相关的其他AOSP树的列表，请查看附录E，例如由Linaro和CyanogenMod维护的那些。有趣的是，大多数这些
替代树也依赖repo，这更是学习如何掌握这个工具的原因。