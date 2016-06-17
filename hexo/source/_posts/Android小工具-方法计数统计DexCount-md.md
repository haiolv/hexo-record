---
title: Android小工具--方法计数统计DexCount.md
date: 2016-06-17 11:59:52
tags: [Android,DexCount,Apk方法统计]
categories: [编程,Android] # 配置categories
---

## Android小工具--方法计数统计DexCount

download:[Android小工具--方法计数统计DexCount 下载](https://github.com/haiolv/tool-android "https://github.com/haiolv/tool-android")

### 目录结构

	.
	├── dex-method-counts.jar
	├── tools.bat
	├── app-debug_dexcount.txt


	**dex-method-counts.jar**
	获得方法的可执行jar包

	**tools.bat**
	tools.bat：获取某一apk方法数的bat批处理命令


### 工具使用

将某一个需要查看方法数的apk拖入tools.bat上，然后就会在 当前目录生成 一个名为 "${apk名称}_dexcount.txt"文件，打开该文件，即可以看到具体的方法分布。

such as:

	Processing D:\UserProfiles\Haiolv\Downloads\app-debug.apk
	Read in 64836 method IDs.
	<root>: 64836
	    <default>: 1
	    adStats: 12
	    android: 16841
	        accessibilityservice: 6
	        accounts: 5
	        animation: 12
	        app: 349
	        bluetooth: 3
	        content: 338
	.....

如上，当将 app-debug.apk  拖入 tools.bat命令后，会在当前目录生成一个 名为 app-debug_dexcount.txt文件，打看我们可以看到该apk先总含有方法数为  64836，bomm..快超过 单个dex所允许的最大方法数了（65536），关于当apk方法数超过Short限制时，请查看 [Configure Apps with Over 64K Methods](https://developer.android.com/studio/build/multidex.html "https://developer.android.com/studio/build/multidex.html")


### 批处理命令详解

	echo 拖入apk包
	REM echo "%~f1"
	REM echo "%~dp0"
	REM echo %0
	pushd %~dp0 
	
	if exist ./dexcount.txt (
		rmdir /s /q ./dexcount.txt
	)
	
	REM echo %~n1
	REM echo %~n1%_dexcount.txt
	REM pause
	
	java -jar ./dex-method-counts.jar "%~f1" > ./%~n1%_dexcount.txt
	popd
