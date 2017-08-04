> 翻译：[neil-xiang](https://github.com/neil-xiang)
> 校对：

## AOSP 初探

	现在你对AOSP有了大致了解，现在开始着手AOSP的开发了。我们现在就从怎样从http://android.git.kernel.org/ 来获取AOSP开始。在开始编译和运行ASOP之前，我们需要花点时间探索AOSP的内容，以及解释在前面几章的内容在AOSP中体现的。本章的最后，我们还将介绍两个在各种平台上的使用的非常重要的工具，`adb`工具和模拟器。

	总的来说，这章内容比较有意思。ASOP是一个令人兴奋并具有大量创新的软件系统。当然，我也承认它不能担任所有角色以及难免会有一些粗糙的部分。但它仍然不乏惊艳的片断。显然，最棒的是我们可以下载它，修改它，出售基于它的定制化产品。所以，卷起你的袖子，让我们开始吧。

##开发主机设置(Development Host Setup)
	就像我们在“Development Setup and Tools“中讨论的那样，你需要一个基于Ubuntu的电脑来运行AOSP。虽然其他的系统也可以工作，但是这个是google的文档中支持的系统。我建议你翻回去重新读一下那一章温习一下基本的AOSP主机需求。并且我也建议你看一下Google的网站‘http://source.android.com’的”Initializing a Build Environment“这一段来得到如何构建你的主机来构建Android 源码的最新要求。该页面还包括配置‘udev’以确保正确设置权限，以便您访问连接到主机Android设备。

