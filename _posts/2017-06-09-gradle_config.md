---
title: 'android gradle加载配置文件参数'
layout: post
tags:
    - android 
    - gradle
    - blog
---
此文只是简单的通过配置变量的方式实现配置信息之间的切换，除去了频繁更改地址、开关、AppKey等其它的配置信息。
<!--more-->

本篇文章简单的使用了gradle的生命周期，有兴趣的童鞋可以自行百度了解一下。

一般android studio刚创建的project的setting.gradle文件只包括一行：  


这一行的意思是构建mutiProject，按照正常的逻辑来说，其实android studio创建的每个module都算是一个单独的Project，而gradle通过构建此文件，将这些项目统一构建为mutiProject。

gradle的构建过程简单理解如下所示:


setting.gradle文件只有一份，这也就意味着整个mutiProject的gradle对象只有一份，它可以在任何project的gradle文件当中获取到这个对象。

build.gradle文件有多份，每个module下存在于单个build.gradle文件，这就意味着，每个module都具备单独的project对象，这就意味着每个module都是一个单独的project。

setting.gradle文件对应着gradle对象的初始化，那一般的初始化操作可以在此gradle脚本文件当中运行，但不要原始生成的include标签抹去，这是gradle构建mutiProject关键所在。

首先，我在项目当中配置了debug.properties以及release.properties,这两个文件：


第二步，在gradle.properties文件当中定义一个boolean值IS_RELEASE_MODE，这个变量就是加载配置了debug和release两个版本的配置文件的开关，如果有多种配置信息的，请改用字符串定义变量。，true表示Release参数，false代表debug参数。


注：此处不得不说一下，gradle.properties文件当中的属性，我暂且这样理解为，在setting.gradle文件初始化gradle对象的时候，这个文件当中的属性已经默认加载到这个gradle对象当中，所以这个文件当中定义的属性是可以直接调用。

第三步，setting.gradle当中读取文件当中定义属性代码展示如下（请配合参考groovy语言，$符是脚本语言的通用处理符号）


同时，如果想后期读取自己定义的properties文件，这时候就需要用来gradle对象的扩展属性，对于gradle的扩展属性是通过gradle.ext标识符来完成的，特别声明，gradle.ext.属性名只需要在第一次赋值的时候需要带上ext标签，在后面的过程可以直接gradle.属性名进行任意的更改和取值。


调用读取配置文件的方法


基本参数配置完毕，可以通过gradle指令（gradlew clean指令就可观察对应扩展属性的值）在IS_RELEASE_MODE=true的情况下，输出如下所示。


如果诸位童鞋觉得这个指令等待时间过长，那就自定义一个task，运行指令如上图所示，task的代码块是定义在project 类当中，所以只能在build.gradle文件当中进行配置：


以上gradle的ext扩展属性对应配置脚本信息已经完毕。而这里需要解释的是上文当中有两种读取配置文件属性的处理方式，下文会具体指出其中需要注意的地方。

第四步，在依赖的最顶层的libray 当中的build.gradle中做如下配置，为什么说要在最顶层的libray当中做这个配置，这是由于每个project（也就是as当中的module）都会在build当中生成对应自己build.gradle当中配置的BuildConfig.java类，所以，把这些配置信息放到最顶层的库，是为了其它依赖者也能够公用这个库的BuildConfig.java类。具体配置如下：


gradle编译完毕之后会在当前build.gradle对应的project的build目录下生成BuildConfig类：


同时，在gradle.properties当中将IS_RELEASE_MODE=false，BuildConfig编译完毕之后就能完成debug.properties文件属性值的切换。


到了这个地方，按照以上步骤走，基本上可以解决多个版本配置信息之间的切换了。但上面提到的两种属性处理方式，在这里，我就通过未定义到BuildConfig的TEST7 属性来说明具体原因，这个点具体如下所示，如果我将TEST5采用非字符串的处理方式，BuildConfig如下所示：


这就导致BuildConfig.java文件默认报错，我想大家还有一种比较简单的处理方式，那就是在配置文件当中直接对需要加在字符串的直接加上字符串就行了，我为了统一性，采用代码控制了这个问题，为了配置文件的属性的统一性。

下面单独说下这个TEST7属性的处理方式：在第三方sdk当中经常有<meta-data />标签形式定义的APP_KEY模式，这个就可以通过android 代码块中的maniPlaceHolder实现。


manifestPlaceHolders可以在androidManifest.xml文件当中读取定义的属性

Manifest文件通过manifestPlaceHolders获取gradle定义的属性
我想大家已经get到这个TEST7使用的点了，对，没错，它一般用于和AndroidManifest.xml完成变量的交互，它采用非字符串读取方式的处理方式，那是因为在默认加载到AndroidManifest.xml中的这个变量会被处理为字符串格式，也可以简单理解，系统内部已经做了字符串的添加操作，这时候再来一次的话，就会导致清单文件的这个变量读取成带字符串的值，这会导致第三方SDK或其它情况的错误。（请注意，有兴趣的同学可以按照字符串方式处理，然后反编译，可以很清楚的在清单配置文件当中看到这个变量的value值。）

demo指向，能用代码说话，就别说废话了--https://github.com/kaycool/EnvironmentConfigDemo