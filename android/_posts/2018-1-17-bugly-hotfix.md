---
layout: post
title: "Tinker 热更新笔记"
modified: 2018-1-17 19:46:00
excerpt: "相关配置、多 Flavor 打包过程、遇到的 BUG..."
tags: [Android]
comments: true
---


- [Bugly Android 热更新使用指南](https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20171123163535)

- [教学视频](http://v.qq.com/boke/gplay/9f3b4b1232819f453becd2356a3493c4_bme000301803d13_5_w0384j4xrnd.html)


### 一、注意事项

- 不能热更新 Manifest 清单文件中的内容
- ...


### 二、相关配置

- enableProxyApplication = true 情况下的配置

	执行 Gradle -> :app -> Tasks -> build -> assemableNone 后，会在 /app/build/bakApk/baseApkDir/none 目录下生成 APK
	
	检查 APK 的 manifest 文件，ApplicationName 会被替换，并生成几个额外的 meta-data，其中包括 TINKER_ID 和 TINKER_PATCH_APPLICATION
	
	- TINKER_ID = noneRelease_base-1.54
	- TINKER_PATCH_APPLICATION = xxx.xxx.CustomApplication



### 三、打包过程

- 生成基包

	1. 修改 TinkerId 为 base-version
	2. 生成包 `Gradle Projects -> :app -> Tasks -> build -> assembleRelease`
	3. 获取 APK，在 `/app/build/bakApk/baseApkDir/flavors/app-flavor-release.apk`

	
- 生成补丁包

	1. 修改代码
	2. 修改 TinkerId 为 patch-version-index
	3. 设置 `baseApkDir` 路径为基包的名称
	4. 生成基包 `Gradle Projects -> :app -> Tasks -> tinker-support -> buildAllFlavorsTinkerPatchRelease`
	5. 获取 APK，在 `/app/build/output/patch/flavors/release/patch_signed_7zip.apk`


- 其它配置

	- buildAllFlavorsDir 构建多渠道补丁时使用
	- isProtectedApp 是否启用加固模式，默认为false
	- enableProxyApplication 是否开启反射Application模式



### 四、遇到的 BUG

- 证书

	```
	Execution failed for task ':app:tinkerPatchNoneRelease'.
	> Could not resolve all dependencies for configuration ':app:sevenZipToolsLocator'.
	   > Could not download SevenZip-osx-x86_64.exe (com.tencent.mm:SevenZip:1.1.10)
	      > Could not get resource 'https://jcenter.bintray.com/com/tencent/mm/SevenZip/1.1.10/SevenZip-1.1.10-osx-x86_64.exe'.
	         > Could not GET 'https://jcenter.bintray.com/com/tencent/mm/SevenZip/1.1.10/SevenZip-1.1.10-osx-x86_64.exe'.
	            > peer not authenticated
	```
	
	原因：电脑上没有 SevenZip 签名软件
	
	解决：设置 path 路径
	
	```
	sevenZip {
        zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
        path = "/usr/local/bin/7za"
    }
	```


- BuglyFileProvider 文件冲突，应用无法安装

	```
	Failed to install xxx.apk: Failure
	[INSTALL_FAILED_CONFLICTING_PROVIDER: Package couldn't be installed in /data/app/com.xxx: 
	Can't install because provider name com.tencent.bugly.beta.fileProvider 
	(in package com.xxx.xxx) is already used by com.xxx.bbb]
	```
	
	原因：android:authorities 不能与已安装应用相同导致。
	
	在配置清单文件时没有添加 com.tencent.bugly.beta.fileProvider，导致自动生成了一个，并且 android:authorities="com.tencent.bugly.beta.fileProvider"。
	
	
	解决：加上 BuglyFileProvider，代码如下：
	
	```xml
    <provider
        android:name="com.tencent.bugly.beta.utils.BuglyFileProvider"
        android:authorities="${applicationId}.fileProvider"
        android:exported="false"
        android:grantUriPermissions="true"
        tools:replace="name,authorities,exported,grantUriPermissions">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/provider_paths"
            tools:replace="name,resource"/>
    </provider>
	```
	
	
### 五、使用 Python 编写脚本提取相关信息（不重要）

因为同时要为好多个应用添加 Bugly 的热更新，而且有四个渠道的安装包，手动将每个文件复制出来并重命名非常的繁琐。借此写一个脚本，顺便练练手（之前只看过文档）:


```
import os
import shutil
import time

BASE_PROJECT_BUILD_DIR = "/Users/ionesmile/Documents/iOnesmileDocs/WorkSpace/Snaillove/ColorLampChangda/app/build"

BAK_PATH = BASE_PROJECT_BUILD_DIR + "/bakApk/"

baseApkDir = "app-0206-18-36-11"

FLAVORS_ALIAS_MAP = {"self360": "360", "selfBaidu": "baidu"}


def getApkVersionCode(baseApk):
    return "1.38"


def getProjectName():
    return "i-lamp"


def copyBaseApkRenameToPath(outPath):
    outPath = os.path.join(outPath, "base")
    if not os.path.exists(outPath):
        os.makedirs(outPath)
    for flavorApk in os.listdir(BAK_PATH + baseApkDir):
        if os.path.isdir(os.path.join(BAK_PATH + baseApkDir, flavorApk)):
            baseApk = os.path.join(BAK_PATH + baseApkDir, flavorApk + "/app-"+flavorApk+"-release.apk")
            print "baseApk", baseApk
            if os.path.exists(baseApk):
                if not FLAVORS_ALIAS_MAP.get(flavorApk, None) is None:
                    flavorApk = FLAVORS_ALIAS_MAP.get(flavorApk, None)
                out_file = os.path.join(outPath, "android_"+getProjectName()+"_v"+getApkVersionCode(baseApk)+"_"+getDateYyyyMMdd()+"_"+flavorApk+".apk")
                print "out_file", out_file
                shutil.copyfile(baseApk, out_file)


def copyBaseFileRenameToPath(outPath):
    outPath = os.path.join(outPath, "baseFile")
    if not os.path.exists(outPath):
        os.makedirs(outPath)

    for flavorApk in os.listdir(BAK_PATH + baseApkDir):
        if os.path.isdir(os.path.join(BAK_PATH + baseApkDir, flavorApk)):
            baseApk = os.path.join(BAK_PATH + baseApkDir, flavorApk)

            out_flavor_path = os.path.join(outPath, flavorApk)
            if not os.path.exists(out_flavor_path):
                os.makedirs(out_flavor_path)

            for child_file in os.listdir(baseApk):
                item_file = os.path.join(baseApk, child_file)
                if os.path.isfile(item_file) and not item_file.endswith(".apk"):
                    out_file = os.path.join(out_flavor_path, child_file)
                    print "item_file", item_file
                    print "out_file", out_file
                    print ""
                    shutil.copyfile(item_file, out_file)



def copyPatchApkRenameToPath(outPath):
    outPath = os.path.join(outPath, "patch")
    if not os.path.exists(outPath):
        os.makedirs(outPath)

    patchPath = os.path.join(BASE_PROJECT_BUILD_DIR, "outputs/patch")
    for flavorApk in os.listdir(patchPath):
        if os.path.isdir(os.path.join(patchPath, flavorApk)):
            baseApk = os.path.join(patchPath, flavorApk + "/release/patch_signed_7zip.apk")
            print "baseApk", baseApk
            if os.path.exists(baseApk):
                if not FLAVORS_ALIAS_MAP.get(flavorApk, None) is None:
                    flavorApk = FLAVORS_ALIAS_MAP.get(flavorApk, None)
                out_file = os.path.join(outPath, flavorApk + "_patch_"+getApkVersionCode(baseApk)+"_7zip.apk")
                print "out_file", out_file
                shutil.copyfile(baseApk, out_file)


def getDateYyyyMMdd():
    return time.strftime("%Y%m%d", time.localtime(time.time()))


# 复制基包和 R 等文件到指定的目录
# copyBaseApkRenameToPath("/Users/ionesmile/Desktop/iLamp")
# copyBaseFileRenameToPath("/Users/ionesmile/Desktop/iLamp")

# 提取补丁包到指定目录
copyPatchApkRenameToPath("/Users/ionesmile/Desktop/iLamp")
```







