---
title: 使用React-Native调用原生模块
catalog: true
date: 2018-12-22 13:11:37
subtitle: React-Native项目
header-img: "code.jpg"
tags:
- Site
- Blog
- Code
catagories:
- Hexo
---

>今年年初因为工作调动去了常州，上了 React 项目，大概做完以后又需要做一个 React-Native App，一边学一边看就上了项目。年后回到南京，这边也需要做一个 React-Native 项目。正好赶上我会一点，就上了，在搭完基本构架，画出页面以后，终于碰到了难题。这个 App 主要功能是实现语音对讲功能，是上汽内部员工使用的对讲 App。对于原生模块，App 与原生语音的交互需要我来完成。这篇博客就提供了一个基础的原生模块交互的过程。

# 搭建项目
在配置完基本的环境以后，可以使用两种方法进行脚手架的搭建，具体操作指令可以移步[这里](https://juejin.im/post/5ae43adaf265da0b851c9d6e#heading-1)
项目搭建完以后，可以在虚拟机或者真机上完美运行，具体配置操作可以查看上一篇博客。

# 创建模块
在项目跑完以后，在主页面添加一个 Button 事件，然后打开文件目录 android⁩/app⁩/src⁩/main⁩/java⁩/cn⁩/retailsolution⁩/console⁩/your-app-name/，
创建一个新的 Java 类：ToastModule.java，具体代码如下：
```
// ToastModule.java

package com.your-app-name;

import android.widget.Toast;

import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

import java.util.Map;
import java.util.HashMap;

public class ToastModule extends ReactContextBaseJavaModule {

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";

  public ToastModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }
}
```

ReactContextBaseJavaModule要求派生类实现getName方法。这个函数用于返回一个字符串名字，这个名字在 JavaScript 端标记这个模块。这里我们把这个模块叫做ToastExample，这样就可以在 JavaScript 中通过NativeModules.ToastExample访问到这个模块。
所以在以上代码的最后加上如下代码：
```
@Override
  public String getName() {
    return "ToastExample";
  }
```

一个可选的方法getContants返回了需要导出给 JavaScript 使用的常量。它并不一定需要实现，但在定义一些可以被 JavaScript 同步访问到的预定义的值时非常有用。
所以在以上代码的最后加上如下代码：
```
@Override
  public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }
```

要导出一个方法给 JavaScript 使用，Java 方法需要使用注解@ReactMethod。方法的返回类型必须为void。React Native 的跨语言访问是异步进行的，所以想要给 JavaScript 返回一个值的唯一办法是使用回调函数或者发送事件.
所以在以上代码的最后加上如下代码：
```
@ReactMethod
  public void show(String message, int duration) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
  }
```

本页面整体代码如下：
```
// ToastModule.java

package com.your-app-name;

import android.widget.Toast;

import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

import java.util.Map;
import java.util.HashMap;

public class ToastModule extends ReactContextBaseJavaModule {

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";

  public ToastModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  @Override
  public String getName() {
    return "ToastExample";
  }

  @Override
  public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }

  @ReactMethod
  public void show(String message, int duration) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
  }
}
```

# 注册模块
创建一个新的 Java 类并命名为CustomToastPackage.java，放置到android⁩/app⁩/src⁩/main⁩/java⁩/cn⁩/retailsolution⁩/console⁩/your-app-name/目录下。具体代码如下：
```
// CustomToastPackage.java

package com.your-app-name;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CustomToastPackage implements ReactPackage {

  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(
                              ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();

    modules.add(new ToastModule(reactContext));

    return modules;
  }

}
```

这个 package 需要在MainApplication.java文件的getPackages方法中提供。所以在目录为：android⁩/app⁩/src⁩/main⁩/java⁩/cn⁩/retailsolution⁩/console⁩/your-app-name/MainApplication.java的文件中添加代码：
```
import com.your-app-name.CustomToastPackage; // <-- 引入你自己的包
...
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
            new CustomToastPackage()); // <-- 添加这一行，类名替换成你的Package类的名字.
}
```

为了让你的功能从 JavaScript 端访问起来更为方便，通常我们都会把原生模块封装成一个 JavaScript 模块。所以在你的src文件夹下创建文件：ToastExample.js。添加如下代码：
```
import { NativeModules } from "react-native";
// 下一句中的ToastExample即对应上文
// public String getName()中返回的字符串
module.exports = NativeModules.ToastExample;
```

# 使用
在刚刚创建的 Button 事件中触发，先引入封装的模块，然后触发方式如下：
```
import ToastExample from "./ToastExample";
...
ToastExample.show("Awesome", ToastExample.SHORT);
```
