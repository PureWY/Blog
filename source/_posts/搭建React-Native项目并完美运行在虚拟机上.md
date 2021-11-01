---
title: 搭建React-Native项目并完美运行在虚拟机上
catalog: true
date: 2018-12-12 15:12:05
subtitle: React-Native项目
header-img: "code.jpg"
tags:
- Site
- Blog
- Code
catagories:
- Hexo
---

>最近公司在准备一个 React-Native 的 App 项目，所以我就准备研究一下 React-Native,因为需要在安卓机上或者安卓虚拟机运行，就涉及到安装Android Studio等等，这玩意需要安装很多东西，初学者很难成功的将项目跑起来。也是为了记录一下自己鼓捣的过程，就写了这样一篇博客，教大家如何完美的将项目运行起来。

>本例以 macOS 操作系统作为开发平台，目标平台是 Android。

# 安装依赖

首先安装必备的Node,并要求版本是 V8.3以上。本人通过 brew 命令行安装，你也可以去官网下载安装包自行安装。具体地址在[这里](https://nodejs.org/zh-cn/)。brew安装指令为：
`brew install node`

React-Native 需要 Java JDK，并且要求版本是1.8，版本不对的去[官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载安装。安装完成以后可以通过指令查看版本号是否正确：
`javac -version`

以上就是基本的依赖环境的安装。

# Android 开发环境安装

Android 开发环境的安装是有些繁琐的，不过只要一步步的来，应该不会出现什么问题。首先我们需要安装  Android Studio，下载地址在[这里](https://developer.android.com/studio/)，注意，此处需要一些科学上网工具，不然是打不开页面的。

下载完成以后点击安装，然后就一直 Next，如果你碰到这个页面，提示无法访问Android SDK，可以暂时不管，直接cancel。
![安装界面](https://purewy.github.io/img/android/android.png)

弹出如下界面，直接next。
![安装界面](https://purewy.github.io/img/android/android1.png)

等到这个界面，选择标准模式，点击Next:
![安装界面](https://purewy.github.io/img/android/android2.png)

然后就是选择界面样式，根据喜好选择以后点击Next。
然后需要我们下载一堆组件，点击Next就会下载，等待下载完成以后就完成了Android Studio的安装。
![安装界面](https://purewy.github.io/img/android/android3.png)
![安装界面](https://purewy.github.io/img/android/android4.png)

## 安装 Android SDK

Android Studio 默认会安装最新版本的 Android SDK。目前编译 React Native 应用需要的是Android 8.1 (Oreo)版本的 SDK。你可以在Android Studio 的 SDK Manager 中选择安装各版本的 SDK。
你可以在 Android Studio 的欢迎界面中找到 SDK Manager。点击"Configure"，然后就能看到"SDK Manager"。
在 SDK Manager 中选择"SDK Platforms"选项卡，然后在右下角勾选"Show Package Details"。展开Android 8.1 (Oreo)选项，确保勾选了下面这些组件：

```
Android SDK Platform 27
Intel x86 Atom_64 System Image
```

最后点击"Apply"来下载和安装这些组件。

## 配置 Android 环境变量

打开命令行，输入以下指令：

`open .bash_profile`

然后在文件底部添加如下代码：
```
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```
保存以后，输入以下指令：

`source .bash_profile`

这样环境变量就配置完成了。

>如果你不是通过Android Studio 安装的sdk，则其路径可能不同。

# 创建 React-Native 项目

新建一个文件夹，cd 到这个文件夹中，输入以下指令：

`npm install -g react-native-cli`

然后创建项目：

`react-native init MyProject`

然后就会一直跑，等到项目中各种依赖安装完成，到此项目就创建成功了。

# 让项目跑起来

## 在安卓真机上运行项目

>首先保证你的手机已经打开开发者模式。

通过 USB 将你的 Android 机链接到电脑上，保证打开开发者模式，然后执行下面的命令：

`react-native run-android`

这时应该会跑很久，但是不出意外会成功运行，这时候就可以在安卓机上看到你的项目了。不过这样并不方便调试，一旦你修改项目代码，并不能保证手机上的项目热更新，但是这种方法真实感更强。

## 通过虚拟机运行项目

打开 Android Studio,随便新建一个项目，一直 Next，成功进入项目以后找到右上角的这个图标：
![创建虚拟机](https://purewy.github.io/img/android/AndroidStudioAVD.png)

打开它，创建一个虚拟机。点击左下角的 Create Virtual Device,然后你会看到这个：
![创建虚拟机](https://purewy.github.io/img/android/AndroidPhone.png)
你可以根据自己电脑性能选择手机型号，分辨率越高需要的条件就越高。我是选择上面的默认型号，然后点击 Next，你会看到这个：
![创建虚拟机](https://purewy.github.io/img/android/AndroidPhone1.png)
请选择上面指示的版本下载，然后 Next，看到这个：
![创建虚拟机](https://purewy.github.io/img/android/AndroidPhone2.png)
到这里点击右下角的完成就可以了。虚拟机就创建成功了。

然后双击你刚刚创建的虚拟机，就会看到仿真的手机在运行。这时候在你可以将你的项目放进你喜欢的编辑器内，然后运行下面的指令：

`react-native run-android`

不出意外你的项目就会在虚拟机上跑起来，是不是很有趣。

# 保持项目热更新

在开发过程中我们需要一直调试你的项目，但是不能重复的跑项目，这样很浪费时间跟资源。所以我们可以这样做，打开你的虚拟机，在你的项目页面按下：cmd + m,会出来这样一个弹出框：
![弹框](https://purewy.github.io/img/android/tankuang.png)
点击红色指示线的按钮，这样就开启了热更新。

到此为止这样一个 React-Native 项目就成功的在 安卓机或者 虚拟机上跑起来了。