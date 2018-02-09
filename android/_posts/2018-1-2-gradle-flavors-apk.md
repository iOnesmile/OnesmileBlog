---
layout: post
title: "Gradle 的 Flavors 的多渠道打包"
modified: 2018-1-2 18:50:00
excerpt: "根据 flavorCompile 依赖不同更新包..."
tags: [Gradle]
comments: true
---


因为上传不同的市场，需要内置不同的更新包，而 360 和 百度 的更新包在上传时会有冲突，所以需要根据情况来编译对应的更新包。

`build.gradle` 中 Flavor 编译，并设置输出文件名代码如下：

```groovy
apply plugin: 'com.android.application'

android {

    productFlavors {
        none { }
        self { }
        self360 { }
        selfBaidu { }
    }

	// 设置输出文件名称（不必须）
    android.applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def flavorAliasMap = ['self360':'360', 'selfBaidu':'baidu']
            String flavorType = flavorAliasMap.get(productFlavors[0].name) == null ? productFlavors[0].name : flavorAliasMap.get(productFlavors[0].name)
            String fileName = "android_ilight_pro_v${defaultConfig.versionName}_${new Date().format("yyyyMMdd")}_${flavorType}.apk"
            output.outputFile = new File(output.outputFile.parent, fileName)
        }
    }
}

dependencies {
	...
	
	// 使用 flavorCompile 依赖不同库
    selfCompile(name: 'updateSelfLibrary_20171221', ext: 'aar')
    self360Compile(name: 'update360Library_20171221', ext: 'aar')
    selfBaiduCompile(name: 'updateBaiduLibrary_20171221', ext: 'aar')
}

repositories {
    flatDir { dirs 'libs' }

    jcenter()
}
```



`build.gradle` 中签名配置代码如下（不必须）：


```groovy
apply plugin: 'com.android.application'

android {
    
    // 配置签名包
    signingConfigs {
        config {
            keyAlias 'xxx'
            keyPassword 'xxxxxx'
            storeFile file('/Users/ionesmile/Documents/iOnesmileDocs/WorkDoc/keystore')
            storePassword 'xxxxxx'
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.config
        }
    }
}
```

目前代码中使用反射的方式来出发检测版本，如下：

```
/** 动态导入不同的更新库，通过反射方式检查更新 **/
public final static void checkVersionUpdate(Activity activity) {
    final String[] updateClassArr = new String[]{
            "com.snaillove.common.update.DynamicUpdateSelf",
            "com.snaillove.common.update.DynamicUpdateBaidu",
            "com.snaillove.common.update.DynamicUpdate360"
    };
    for (String clazzName : updateClassArr) {
        try {
            Class cls = Class.forName(clazzName);
            Method setMethod = cls.getDeclaredMethod("exec", Activity.class);
            setMethod.invoke(cls.newInstance(), activity);
        } catch (Exception e) {
        }
    }
}
```





