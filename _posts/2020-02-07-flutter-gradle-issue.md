---
layout: article
title: Flutter 关于 Android Gradle 的配置问题
key: flutter-gradle-issue
show_author_profile: true
sharing: true
tags: ["Flutter", "Gradle"]
---

因为 Gradle 的 Maven 仓库在国外, 导致运行 Flutter 项目一直卡在在步骤 `Running Gradle task 'assembleDebug'...` , 本文记录解决此问题的过程.<!--more-->

<div style="margin: 0 auto;" align="justify" markdown="1">

## 前言

Gradle 是 Android 的构建工具,它可以帮助管理项目中的差异,以来,编译,打包,部署等等[^gradle],类似于 iOS 的构建工具 Cocoapods.

新建的 Flutter 工程会在 android 目录下生成 `build.gradle` 配置文件.同时,Flutter项目android工程app主模块还引入了 `flutter.gradle` 配置文件,该文件在 Flutter sdk 的 `flutter/packages/flutter_tools/gradle/` 目录下,目的是在普通android工程的编译打包流程中插入一些Flutter任务.

```bash
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"
```

在app主模块，当Gradle运行到 `apply from` 时,进入flutter.gradle执行代码流程.

<div align="center">
<img src="https://alain-hsu-blog.oss-cn-shenzhen.aliyuncs.com/apply-from-flutter-gradle.webp" alt="topo" width="500px" class="shadow rounded">
</div>

## 科学上网条件下的解决方案

将代理端口添加到 `gradle.properties` 中.

代理端口在 Mac 的 System Preferences > Network > Advanced... > Proxies 可以看到.

<div align="center">
<img src="https://alain-hsu-blog.oss-cn-shenzhen.aliyuncs.com/socks-proxy.png" alt="topo" width="500px" class="shadow rounded">
</div>

`gradle.properties` 在 Mac 上通常位于用户根目录下的 `.gradle` 隐藏文件夹中. 如下图所示添加代理端口.

<div align="center">
<img src="https://alain-hsu-blog.oss-cn-shenzhen.aliyuncs.com/gradle-properties.png" alt="topo" width="500px" class="shadow rounded">
</div>

## 使用国内源的解决方案

需要修改 gradle 仓库地址,改用阿里云代理. 需要修改两个地方, Flutter SDK 目录下的`flutter.gradle` 和自己 Flutter 工程android 目录下的 `build.gradle`, 前者只需要修改一次, 后者在每次新建工程的时候都需要修改. 关于这两个文件的描述可以看[前言](#section).

* 修改 `flutter.gradle`

<div align="center">
<img src="https://alain-hsu-blog.oss-cn-shenzhen.aliyuncs.com/flutter-gradle.png" alt="topo" width="500px" class="shadow rounded">
</div>

* 修改 `build.gradle`

<div align="center">
<img src="https://alain-hsu-blog.oss-cn-shenzhen.aliyuncs.com/build-gradle.png" alt="topo" width="500px" class="shadow rounded">
</div>

### 遇到新问题

#### 500 网络错误

```bash
Received status code 500 from server: Internal Privoxy Error
```
* **原因**: 
  在科学上网时添加了代理端口到 `gradle.properties` 中,停止科学上网后代理并不会自动删除,此时仍指向代理,从而导致访问失败.

* **解决方案**: 
  手动删除 `gradle.properties` 中添加的代理.

</div>

[^gradle]: [如何通俗地理解 Gradle？ - ghui的回答 - 知乎](https://www.zhihu.com/question/30432152/answer/48239946)
