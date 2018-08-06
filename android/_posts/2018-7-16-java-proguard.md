---
layout: post
title: "Android 使用 Proguard 混淆"
modified: 2018-7-16 19:30:00
excerpt: "基本功能，规则说明，使用技巧..."
tags: [优化]
comments: true
---


ProGuard 是开源的优化 Java 字节码工具。官方称可用减少 10% 体积，并提升 20% 运行效率。将类名、方法名、变量名混淆成a、b、c基本字母，一定程度上提高了反编译的难度。

- 压缩（Shrinking）：从入口开始建立引用关系网，去除网外为使用的代码。

- 优化（Optimization）：对入口点以外所有的方法进行分析，将其中一部分方法变为 final的，static的，private的或内联的，从而提高执行效率。

- 混淆（Obfuscation）：将入口点以外的类、方法、成员重构为简短的名字，可以减小生成文件体积，同时混淆代码。


#### 官网手册： [Proguard Manual Usage](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/usage.html)


### 一、Android 中配置

- 在 build.gradle 中的配置

	```
	buildTypes {
	    release {
	        minifyEnabled true
	        proguardFile getDefaultProguardFile('proguard-android.txt')
	        proguardFile 'proguard-rules.pro'
	    }
	}
	```
	
	- `proguard-android.txt` 是官方提供的通用混淆配置，文件路径在 `\sdk\tools\proguard\proguard-android.txt` 下
	- `proguard-rules.pro` 即自定义配置文件

	

### 二、混淆规则

- keep 的使用
	- `keep [,modifier,…] class_specification` 保留指定的类名以及成员
	- `keepclassmembers [,modifier,…] class_specification` 留住成员而不能保留住类名
	- `keepclasseswithmembers [,modifier,…] class_specification` 可以根据成员找到满足条件的所有类而不用指定类名,可以保留类名和成员名

- modifier 可选值

	- `allowshrinking` 允许其被压缩,就是说指定的内容有可能被移除,但是如果没有被移除的话它也不会在后续过程中被优化或者混淆.
	- `allowoptimization` 允许其被优化,但是不会被移除或者混淆(使用情况较少)
	- `allowobfuscation` 允许其被混淆,但是不会被移除或者优化(使用情况较少)

- class_specification 规则

	是类和成员的一种模板,只有符合此模板的类和成员才会被应用keep规则。
	
	- 指定类
		- `class` 可以指代任意类和接口,interface指明为接口,enum指明为枚举
		- `classname` 必须用全名,例如 `java.lang.String`，也可以使用正则表达式`（?|\*|\*\*）`
		- `extends` 和 `implements` 关键字是等价的
		- `@` 指明类或成员具有某些注解

	- 指定成员
		- `\<init\>` 代表任意构造方法.
		- `\<fields\>` 代表任意域.
		- `\<methods\>` 代表任意方法.
		- `*` 代表任意成员(包括成员变量和方法).
		- 类型描述通配符
			- `%` 表示任意基本类型(int,char等,但是不包括void).
			- `?` 表示类名中的任意单个字符.
			- `*` 表示类名中的任意多个字符,不包括分隔符(.).
			- `**` 表示类名中的任意多个字符,包括分隔符(.).
			- `***` 表示任意类型.
			- `...` 表示任意多个任意类型的参数.


### 三、Android 中不要混淆的类

- Parcelable 中的 Creator 成员
	
- Serializable 的多数子类
	- 并不是所有的子类都不能混淆：只短暂的保持数据，对新版本不会有影响的不需要混淆（如 Bundle 的数据，另外相比于 Parcelable，尽量少用 Serializable）
		
- JsonBean 对象
	- 可以使用 Gson 中的 `@SerializedName("xxx")` 给属性添加注释

- AIDL 接口的类名

- 另外通用的（native、R资源、四大组件、view等）


### 四、Log 日志的优化

- 当 message 的内容没有计算时(追加、调用方法等)，可以直接调用打印
- 当 message 有计算时，建议使用如下方式
	
	```java
	if (BuildConfig.DEBUG){
	    Log.v(TAG, "xxx = " + xxx + "   method = " + method(xxx));
	}
	```
	
	- 当打正式包时 `BuildConfig.DEBUG` 的值为 false，if 条件中的内容不可能执行，ProGuard 的优化会删除该代码。
	- 如果把这句话封装在方法中，再调用方法。由于在调用方法前先要执行计算操作，虽然不打印但是会多一次无意义的计算


### 五、使用 `@NoProguard` 注释
	
为了方便灵活的配置（类、方法、成员属性）混淆，可以通过定义一个注释标识，让所有引用了该注释的对象都不会混淆。

定义注释：
	
```java
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.FIELD})
public @interface NotProguard {
}
```
	
配置使用了该注释的代码不混淆
	
```proguard
# 配置使用了 @NotProguard 注释的类不混淆
-keep @xxx.xxx.NotProguard class * {*;}
# 配置使用了 @NotProguard 注释的成员变量不混淆
-keepclassmembers class * {
@xxx.xxx.NotProguard <fields>;
}
# 配置使用了 @NotProguard 注释的方法不混淆
-keepclassmembers class * {
@xxx.xxx.NotProguard <methods>;
}
```


### 六、使用经验补充

> 使用版本：     
> Gradle: gradle-4.1-all      
> android.build: 2.3.2

- 查看最终的混淆配置文件，如下代码配置输出路径

	```
	-printconfiguration "build/outputs/mapping/configuration.txt"
	```

	- 自动生成了 layout 中被引用 View 的构造函数不会混淆的规则

		```
		# view res/layout/fmxos_fragment_pay_track_list.xml #generated:14
		-keep class com.fmxos.platform.ui.view.RichTextView {
		    <init>(...);
		}
		```
		
		因此让继承了 View 的类不被混淆的规则是多余的，在代码中被引用的 View 依然可以混淆。
		
		
	- 自动生成了使用 `@Keep` 不被混淆的规则，替代了前面提到的 `@NotProguard`

		```
		# Understand the @Keep support annotation.
		-keep class android.support.annotation.Keep
		
		-keep @android.support.annotation.Keep class * {
		    <fields>;
		    <methods>;
		}
		
		-keepclasseswithmembers class * {
		    @android.support.annotation.Keep
		    <methods>;
		}
		
		-keepclasseswithmembers class * {
		    @android.support.annotation.Keep
		    <fields>;
		}
		
		-keepclasseswithmembers class * {
		    @android.support.annotation.Keep
		    <init>(...);
		}
		```

	- 自动生成了 `AndroidManifest` 文件中注册的四大组件类名不混淆规则

		```
		# view AndroidManifest.xml #generated:35
		-keep class com.fmxos.platform.ui.activity.MusicPlayerActivity {
		    <init>(...);
		}
		```

	- 自动生成了 xml 文件中控件使用了 onclick 属性的方法不混淆规则

		```
		# onClick res/layout/fmxos_patch_music_player_control.xml #generated:8
		-keepclassmembers class * {
		    *** onClick(...);
		}
		```
		
	- 其它...

	
- Retrofit 的 service 接口定义的参数被 Shrinking，导致参数被删除。可以用 `-keepclassmembers` 保持这些参数。



