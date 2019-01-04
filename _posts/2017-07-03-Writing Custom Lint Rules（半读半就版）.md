---
title: 'Writing Custom Lint Rules（半读半就版）'
layout: post
tags:
    - android 
    - lint
    - google翻译
---
[官方原文链接地址](http://tools.android.com/tips/lint-custom-rules)
本文只是个人为了整理Lint相关知识，依据谷歌翻译整理而成，水平确实不堪，很多地方自己都读不太懂，见谅...
<!--more-->

新的2015年8月27日：现在有一个最新的示例项目，显示了如何编写自定义lint检查-包括单元测试
[https://github.com/googlesamples/android-custom-lint-rules](https://github.com/googlesamples/android-custom-lint-rules)
该代码库编写一个lint规则比下面写上去包含的代码有更好的基础。

Lint 在ADT20上预先配置了大概100中检查。但是，也可以使用附加规则进行扩展。例如，如果你是库项目的作者，并且你的库项目具有一定的使用要求，则可以编写额外的lint规则去检查你的库是否正确使用，然后你可以向这个库项目的用户分发这些额外的lint规则。同样，您可能需要执行公司本地的规则。

本文介绍如何编写自定义lint规则。首先，请参阅Google I/O 2012“开发工具的新功能”中的以下演讲，用一个简短的demo了解如何编写和使用lint规则。（跳转至视频55分钟18秒，如果嵌入式播放器未自动跳转。）
（...................................内嵌视频好高端啊，诸位请自行官网............................................）

以下是该视频的详细信息。

为Android Studio创建自定义规则项目


首先，你需要创建一个单独的java项目去编译这个lint 规则。请注意，这是一个java项目，而不是Android项目，因为此代码将被运行在Android studio，IntelliJ，Eclipse，或Gradle，或命令行lint工具中运行--而不是在一台设备上运行。


最简单的方法是从示例项目开始，解压它，然后根据需要更改源代码。你可以在Android Studio或者IntelliJ通过指向要导入的build.gradle项目.

这是一个Gradle项目，它具有使用Lint APIs的所有依赖项，并使用正确的清单条目构建.jar文件。

为Eclipse创建自定义规则项目
如果你使用Eclipse，使用一个简单的库创建一个简单的java项目。


接下来，添加 lint_api .jar文件到项目的classpath。这个jar文件包含规则实现的Lint APIS，你可以在sdk的安装目录 tools/lib/lint_api.jar中找到lint_api jar包


创建检测器
当你实施“lint规则”时，你将准备实现一个“检测器”，它可以识别一个或多个不同类型的问题。这种分离使得单个检查器识别与逻辑相关的不同类型的问题,但是可能具有不同的严重性，描述，并且用户可能想要独立的抑制。例如，清单检测器查找单独的问题，如不正确的注册顺序，缺失minSdkVersion声明等。有关如何编写lint检查的更多详细信息，请参阅编写Lint Check文档。

在我们的示例中，我们假设我们有一个自定义View，并且我们想要制定一个lint 规则，确保这个自定义View的所有使用处都定义了这个特定的View属性。

这里是完整的检测器的源代码：


首先你可以看到这个检测器继承ResourceXmlDetector。这是一种用于资源xml文件如布局和字符串资源声明的检测器。还有其它类型的检测器，例如处理java 源代码或者字节码的检测器。

getApplicableElements()方法返回一组当前检测器关心的xml标签。这里我们只返回了我们自定义view的标签。lint基础设施将为每次出现的标签调用visitElement()方法--并且仅针对该标记的出现。在visitElement()方法中，我们简单的看当前的元素是否定义了我们自定义的属性（“exampleString”），如果没有，我们就报告一份错误。

visitElement方法传递一个context对象。这个context提供很多相关的context；例如，你可以查找相应的项目（从这儿请求minSdkVersion或者targetSdkVersion），或你可以查找正在被分析的xml文件的路径，或你可以创建一个错误的位置（如此处所示）。在这里，我们传递这个元素，这将使错误指向元素的开头，但是你可以例如传递一个xml属性对象，这将指向一个特定的属性。

再说一次，查看Writing a Lint Check文档以获取更多的细节。

创建问题
report()方法第一个参数是ISSUE。这是该检测器报告问题的引用。这是我们在上面类中顶部定义的静态字段。


一个问题有几个不同的属性，按以上顺序定义：

      ◆id，这是一个与这个问题有关的常量，应该是简短和描述性的；这个id例如用于java抑制注释和
        xml抑制属性来标识这个问题。
      ◆摘要。这应该是问题的简要（单行）摘要，例如在Eclipse中的Lint Options UI和其它简要描述
        这个问题的地方。
      ◆说明。这是对问题的更长的说明，这应该向lint的用户解释这是什么问题。通常一次 lint 的错误
        信息是简短的（单行），并且有时候很难在单行错误信息中完全解释一个细的问 题，所以该说
        明用于提供更多的上下文。该说明在lint命令行工具创建的完整HTML报告中显示，在Eclipse 
        Lint窗口中显示当前选定的问题等。

      ◆类别。这里有许多预定义的类别，类别可以嵌套（例如可用性>图标），并且这有助于 
        用户过滤和排序问题，或者通过命令行仅运行某些类型的问题。最常见的类别    
        是“correctness”和"performance",但是也包括国际化，可访问性，可用性等。
      ◆优先权。优先级是1-10之间的整数，其中10是最重要的，并且这被用于排序相应每一个其
        它优先级的问题。
      ◆严重性。一个问题可以有严重的，错误，警告，或忽略默认的严重性。请注意我所说的默
        认：用户可以通过一份lint.xml 文件（更多信息）覆盖问题所使用的严重性。严重和错误在
        eclipse中作为“错误”展示，但是严重的问题在用户想要在Eclipse当中打包一份APK的任何
        时候是自动运行的（没有用户的介入），并且如果它们中的任何一个发现错误，然后这个
        打包过程将被停止。“忽略”严重性的规则不运行。
      ◆检测器类。这只是指向负责识别问题的检测器，这应该指向我们自己的类。请注意，我们
        通过Lint自动实例化检测器，这是为每次运行完成的。因此，你不需要担心在运行之后清除
        检测器中的实例状态。在Eclipse中（其中Lint可以运行多次），将为每次运行创建一个新
        的检测器。因此，你的lint规则必须有一个公有的默认构造器。（如果没有指定，这个编译
        器将自动为你创建一个，就是上面的例子
      ◆检测范围。这决定了这个问题适用于哪些类型的文件。这里我们只是声明我们应用于xml文
        件，但未使用的资源问题的范围包含java和xml文件。

你还可以在问题上调用其它方法，例如，将问题定义为默认禁用，或设置“更多信息”URL。

我们已经创建了一个问题的实例。内置的lint问题都在BuiltinIssueRegistry类中注册。然而，对于自定义规则，我们需要提供我们自己的注册表。每一个自定义jar文件提供它们自己问题的注册类，其中每个问题注册表可以包含由给定jar文件标识的一个或多个问题。

这是我们问题的自定义注册表：


getIssues()方法返回的是由此注册表提供的问题列表。我们在这种特殊的情况下明显可以使用Collections.singletonList()来代替Arrays.asList()，但是这里通常会有多个问题。请注意，注册表必须具有公共默认构造参数，以便它能够被lint实例化。

注册注册表
最后一件事情我们需要去做的是注册这个问题注册表，以便可以由lint找到。我们通过编辑jar文件的清单文件来做到这一点。

如果你是在Gradle/Android studio项目中使用它，则由Gradle处理；只要打开build.gradle文件并根据需要更新jar条目中注册表的路径。

在Eclipse，创建一个清单文件，如下所示：


然后从你使用了上面清单文件的项目中导出一个jar文件。（我在Eclispe当中遇到了一些困难；导出jar对话框中有明确的选项可以做到这一点，但是当我查看导出的.jar文件并打开其中的manifest文件，它不包括我上面的两行，所以我手动创建jar文件：jar cvfm customrule.jar META-INF/MANIFEST.MF googleio）

注意：Eclipse中的导出程序期望在第二行之后添加换行符。所以，确保最后有一个空行，然后直接从Eclipse中导出应该正常工作。

注册自定义jar文件
现在我们有自己自定义规则.jar文件。当lint运行时，它将在~/.android/lint/目录查找自定义规则jar文件，因此我们需要将其放在那里：


(有人问过这个；Lint基本上调用了一般的Android工具方法来找到“工具设置目录”，用于模拟器快照，ddms配置等。你可以在这里找到相关的代码：https://android.googlesource.com/platform/tools/base/+/master/common/src/main/java/com/android/prefs/AndroidLocation.java)

搜索位置
android目录的位置通常是主目录；lint将在$ANDROID_SDK_HOME，${user.home}(java属性)和在$HOME中搜索，只是为了澄清。Lint按顺序搜索第一个存在的位置：
◆ANDROID_SDK_HOME(系统环境或环境变量)
◆user.home(系统环境)
◆HOME(环境变量)

所以，如果ANDROID_SDK_HOME存在的话，然后user.home和HOME将被忽略！这可能看起来是直观的，但是原因是ANDROID_SDK_HOME并不指向你的SDK安装路径；环境变量为ANDROID_SDK。最近的代码也在寻找ANDROID_HOME。所以，ANDROID_SDK_HOME真的是一个替代的主目录位置，当你指向特定的模拟器AVDs等进行单元测试时，在构建服务器上使用。这个命名是非常不幸的，并且导致了各种错误，但是很难改变，而不会破坏用户过去已经依赖的记录行为。现在，确保你不会使用环境变量ANDROID_SDK_HOME来指向你的sdk安装目录。

之后，它查找子路径/.android/lint/，即其中一个对应于刚找到的位置。
◆ANDROID_SDK_HOME/.android/lint/
◆user.home/.android/lint/
◆HOME/.android/lint/
and loads any JAR files it finds there.

并加载在那里找到的任何jar文件。

因此，如果你在〜/ .android / lint /和ANDROID_SDK_HOME存在时有 Lint jar文件，你的JAR 文件将不会被读取。 

如果你在~/.android/lint/和ANDROID_SDK_HOME不存在时有Lint JAR文件，你的JAR文件将被读取。

如果ANDROID_SDK_HOME存在，将你的Lint JAR 文件放到ANDROID_SDK_HOME/.android/lint/

运行自定义检查

最后，让我们运行lint去确保它知道我们的规则：


如果这不起作用，通过确保你的jar文件在正确的位置来解决这个问题，它包含在清单配置文件中正确注册的行数，完整的包名称和类名称是正确的，它是可实例化的（公有的默认构造器），并注册您的问题。

最后，让我们来测试这条规则，这是一个示例布局文件：


现在让我们在包含上面布局的项目中运行lint（并使用 --check MyId参数，我们仅检查我们自己的问题）：


使用Jenkins /其他构建服务器
如果你作为正在连续构建的一部分运行lint，而不是将自定义lint规则放在构建服务器使用的账户主目录当中，你还可以使用环境变量$ANDROID_LINT_JARS来指向包含要是用lint规则的jar文件的路径（使用File.pathSeparator来分隔，例如在Windows和其它地方）。更多关于Jenkins+Lint的使用信息，请查看https://wiki.jenkins-ci.org/display/JENKINS/Android+Lint+Plugin。

更复杂的规则
上面的检查非常简单。还有更多的可能。一个真正有用的资源指出如何更多的查看存在的100左右的问题。查看SDK git信息库，在sdk/lint/，特别是sdk/lint/dl/lint/libs/lint_checks/src/ com / android / tools / lint / checking。

贡献规则
自定义lint规则支持在lint当中主要用于编写真正的本地规则。如果你想到一般感兴趣的Lint检查，你可以使用上述方法来开发和检查规则。然而，请考虑将规则归因于AOSP，以便所有的Android开发者可以从检查中受益！请参阅合作页。

源代码
在Gradle/Android Studio/IntelliJ中使用

this sample project

和在 Eclipse中使用

this .zip

在Eclipse当中打开项目之前，上述自定义lint规则和 lint_api.jar包复制到项目的根目录。

更多信息
如果你有Google+账号，你可以在此发表评论或提问。https://plus.google.com/u/0/116539451797396019960/posts/iyH3ER3LJF7