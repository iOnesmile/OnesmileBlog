---
layout: post
title: "Android 中使用 Kotlin 快速入门"
modified: 2017-7-31 19:35:00
excerpt: "Kotlin 的对象结构、函数，环境搭建，Anko Layout..."
tags: [Kotlin]
comments: true
---

#### 一、类与方法

1，类

- 类的声明

	```kotlin
	class Bar(var b: Int): Foo() {
	    var c = 1
	    init {
	        println("class initializer")
	    }
		
	    constructor(): this(1) {
	        println("secondary constructor")
	    }
	}
	```
		
	Bar类在这里继承了Foo类，Bar类有两个构造函数，直接在Bar类头的是primary constructor，另外一个构造函数使用constructor关键字定义，注意必须要先调用primary constructor，另外，init标明的是class initializer，每个构造函数都会首先调用class initializer里面的代码，再调用构造函数
	
- 创建类的实例，不需要 new 

	```kotlin
	var bar = Bar()
	```
		
- 继承   

	内定义默认是 final 的，要想能被继承，基类头必须有 open 注解
	
- Inner class

	```kotlin
	class Outer {
	    class Inner {      
	    }
	}
	```
		
	与 Java 不同，Kotlin 中所有的内部类默认就是静态的，这样可以减少很多内存泄露的问题。如果需要在内部类中引用外部类对象，可以在Inner类的声明前加上inner关键字，然后在Inner类中使用标记的this：this@Outer来指向外部类对象
	
- Singleton
	
	```kotlin
	object Single {
	    var c = 1
	
	    fun foo() = println("foo")
	}
	```
		
	单利对象用 object 关键字表示，可以直接使用 Single.foo() 来调用了

2，接口

```kotlin
interface Interface {
    fun foo() {
        println(1)
    }
    fun bar()
}
```
	
可以带有默认的实现方法，并且不允许通过属性来维护状态。

	
3，函数

- 函数声明

	```kotlin
	fun foo(va: Int): Int {
	    return 1
	}
	```
		
- 也可以使用单行声明

	```kotlin
	fun foo(va: Int): Int = 1
	```
		
- 重载
	
	与类的派生一样，允许重载的方法要有open注解，而在派生类中重载时要使用override注解
	
	```kotlin
	override fun foo(va: Int): Int {
	    return 2
	}
	```

4，修饰符

5，成员变量的 Get 与 Set

> 注：   
> 1，类和方法默认定义都是 final，以此来提高效率。类想要被继承用 open 关键字   
> 2，类 和 成员变量 默认是 public 修饰

#### 二、语法

**1，语法糖，对类的扩充**

在不修改类的原始定义的情况下实现对类的扩展，如下面的代码为Person类增加了一个名为isTeenager的扩展：

```kotlin
fun Person.isTeenager(): Boolean {
    return age in 13..19
}
```

**2，[排除空指针](https://zhuanlan.zhihu.com/p/27285806?utm_source=tuicool&utm_medium=referral)**

- 定义一个为空的变量是需要加上 ? 符号

	```kotlin
	var text: String? = null
	```
	
- 操作一个可能为空的对象时，同样要加上 ? 符号

	```kotlin
	var length = text?.length
	```
   		
- 如果将该变量传递给函数，在参数后面需要加 !! 符号

	```kotlin
	if (text != null) {
	    customPrint(text!!)
	}
	```

- 如何去掉 !! 符号呢，当代码充满该符号时显然很不优雅，这时可以使用 let 函数

	```kotlin
	text?.let { customPrint(it) }
	```
		
- 如果遇到多个参数的情况，你可以选择嵌套多个 let，但这样可读性并不好。比如：

	```kotlin
	if (mUserName != null && mPhotoUrl != null) {
	   uploadPhoto(mUserName!!, mPhotoUrl!!)
	}
	```

	这时你可以构建一个全局函数：
	
	```kotlin
	fun <T1, T2> ifNotNull(value1: T1?, value2: T2?, bothNotNull: (T1, T2) -> (Unit)) {
	   if (value1 != null && value2 != null) {
	       bothNotNull(value1, value2)
	   }
	}
	```
		
	调用方式
	
	```kotlin
	ifNotNull(mUserName, mPhotoUrl, {name, url ->
			uploadPhoto(name, url)
	})
	```

**3，高阶函数和Lambda表达式**

- 例如给一个变量赋 lambda 表达式 {x,y->x+y}

	```kotlin
	val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
	```
		
- 定义一个可以传表达式的高阶函数

	```kotlin
	fun doubleTheResult(x:Int, y:Int, f:(Int, Int)->Int): Int {
	    return f(x,y) * 2
	}
	```kotlin
		
- 调用方法如下

	```kotlin
	val result1 = doubleTheResult(3, 4, sumLambda)
	或
	val result2 = doubleTheResult(3, 4, {x,y -> x+y})
	```

**4，范围表达式**

- 范围创建只需要 .. 操作符，为升序，例如：

	```kotlin
	// 该范围包含数值1,2,3,4,5
	val r1 = 1..5
	```
		
- 如果要表示降序，用 downTo 函数
		
	```kotlin
	// 该范围包含数值5,4,3,2,1
	val r2 = 5 downTo 1
	```
		
- 如果步长不是1，则需要使用step函数

	```kotlin
	// 该范围包含数值5,3,1
	val r3 = 5 downTo 1 step 2
	// 同理，升序的序列
	val r4 = 1..10 step 2
	```
		
		
**5，条件结构**

- if 表达式（类似于 Java 的 ?: 运算符）
	
	```kotlin
	var age = 20
	val isEligibleToVote = if(age > 18) "Yes" else "No"
	```
		
- when表达式（类似于 Java 的 switch，但功能更强大）

	```kotlin
	val age = 17
	
	val typeOfPerson = when(age){
	    0 -> "New born"
	    in 1..12 -> "Child"
	    in 13..19 -> "Teenager"
	    else -> "Adult"
	}
	```
	
**6，循环结构**

使用 for..in 遍历数组、集合及其它提供了迭代器的数据结构，语法同Java几乎完全相同，只是用 in 操作符取代了 : 操作符

```kotlin
val names = arrayOf("Jake", "Jill", "Ashley", "Bill")
	
for (name in names) {
    println(name)
}
```
		
while 和 do..while 循环的语法与Java完全相同。
	
**7，字符串模板**

可以在字符串中嵌入**变量**和**表达式**，例如：

```kotlin
val name = "Bob"
println("My name is ${name}") //打印"My name is Bob"
	
val a = 10
val b = 20
println("The sum is ${a+b}") //打印"The sum is 30"
```

#### 三、XML 布局 + kotlin-android-extensions

1, 通常在 xml 中查找控件的写法

```kotlin
val name = find<TextView>(R.id.tv_name)
// 等同于 findViewById()
val name = findViewById(R.id.tv_name) as TextView
name.text="张三"
```
	
2，如果使用扩展后，可以直接调用并赋值

```kotlin
tv_name.text="张三"
```

## 环境配置

1，项目下面的 build.gradle 加入如下代码：

```gradle
buildscript {
	
    ext.kotlin_version ="1.0.4"
    
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```
	
2，app 下面的 build.gradle 加入如下代码：

```groovy
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
	
android {
    sourceSets{
        main.java.srcDirs+='src/main/kotlin'
    }
}
	
dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    compile 'org.jetbrains.anko:anko-sdk25:0.10.0-beta-1'// sdk15, sdk19, sdk21, sdk23 are also available
    compile 'org.jetbrains.anko:anko-appcompat-v7:0.10.0-beta-1'
}
```
	
3，Gradle Sync

4，菜单栏 ---> Code ---> Convert Java File to Kotlin File

## Anko Layout

#### 一、优点

1，[运行速度快](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/1123/6795.html)。XML布局是在运行时解析的，也就是说XML需要从资源文件中获取，然后 XmlPullParser 需要解析所有的元素并一个一个的创建它们。还要解析元素的属性，然后设置，这个过程非常繁重。

2，类型安全，不再需要那么多的 *findById()* 之后的类型转换。

3，null 安全，Kotlin 里，如果一个变量用？表示为可空，并且使用？之后再调用的时候，即使变量为空也不会引发异常。

4，[代码复用](http://www.alliedjeep.com/118149.htm)，可以通过继承AnkoComponent的方式实现代码复用。XML布局是每一个Activity，每一个View各自专属一个，代码复用比较少。


#### 二、缺点

1，Anko DSL 布局不能预览。虽然有一个叫 Anko Preview Plugin 的预览插件，但是每次修改后都需要 make 下才能预览，关键是在新版本 Android Studio2.2 以上都不支持。

- 笔者在 Android studio2.3 上安装该插件，导致重启后无法进入项目界面。
- 幸好在启动页面的左下角有一个 `Config` 选项，点击其中的 `Plugin`，卸载 `Anko Preview` 插件才可以正常启动。

2，定义 id 比较繁琐，需要定义一个变量，或者在 values 资源文件下定义 ids。不用 id 行不行呢？你去问问 RelativeLayout 答应不答应吧。

3，如果定义在 xml 的话，可以直接通过 id 使用对应的 View（XML 布局 + kotlin-android-extensions 的方式），但是在 Anko DSL 布局的话，只能通过定义变量的方式来实现。

4，动态替换外部资源以达到换肤的效果，那么 XML 显然比 Kotlin 代码要来得容易：前者可以编译成一个只有资源的 apk 供应用加载，后者的话就得搞一下动态类加载了。


#### 三、引用方式

```kotlin
// 继承 AnkoComponent 创建布局
class LoginLayout<T> : AnkoComponent<T> {

    override fun createView(ui: AnkoContext<T>): View {
        return with(ui){
            ...
        }
    }
}

// Activity 中引用
override fun onCreate(savedInstanceState: Bundle?) {
    LoginLayout<MainActivity>().setContentView(this)
}

// Fragment 中引用
override fun onCreateView(inflater: LayoutInflater?, container: ViewGroup?, savedInstanceState: Bundle?): View? {
    var view = LoginLayout<LoginFragment>().createView(AnkoContext.create(context, LoginFragment()))
    return view
}
```


#### 四、常用语法

1，定义 TextView

```kotlin
textView("Hello") {
    textSize = 16f
    textColor = Color.RED
    backgroundResource = R.drawable.shape_et_bg
    gravity = Gravity.CENTER
}.lparams(matchParent, wrapContent){
    margin = dip(12)
    padding = dip(2)
}
```

2，提取样式

```kotlin
// 给 EditText 扩展样式方法
fun EditText.commonStyle(){
    textSize = 16f
    backgroundResource = R.drawable.shape_et_bg
}
	
// 直接在布局的后面添加上
.style {
    view ->
    when (view) {
        is Button -> {
            view.gravity = Gravity.CENTER
        }
        is TextView -> {
            view.gravity = Gravity.LEFT
            view.textSize = 20f
            view.textColor = Color.DKGRAY
        }
    }
}
```

3，设置点击事件

```kotlin
var etInput = editText {
    hint = "请输入文字"
    commonStyle()
}
	
button("点我"){
	// 在按钮属性内部设置点击事件
    onClick {
        toast("输入的内容：${etInput.text}")
    }
}
	
// 通过变量 + . 的方式设置
etInput.onClick { 
    
}
```

4，布局方式

```kotlin  
val ID_USERNAME = 1

// 垂直布局，== LinearLayout + orientation="vertical"
verticalLayout {  }
// 相对布局，需要使用到 ID
relativeLayout {
    textView("姓名") {
        id = ID_USERNAME
    }
    textView("描述") {

    }.lparams {
        below(ID_USERNAME)
        alignParentLeft()
    }
}
// 线性布局
linearLayout {
    orientation = LinearLayout.HORIZONTAL
}
frameLayout { }
tableLayout { }
```

5，*ui: AnkoContext<T>* 

```kotlin  
// 包含的变量
val ctx: Context
val owner: T
val view: View
    
// 例如，可以通过 owner 直接调用外部 Activity 的方法
override fun createView(ui: AnkoContext<T>): View {
    if (ui.owner is Activity) {
        (owner as Activity).onBackPressed()
    }
}
```


## 参考链接

Kotlin Primer·第二章·基本语法 <https://kymjs.com/code/2017/02/04/01/>

<http://blog.csdn.net/io_field/article/details/53365834>


只需五分钟，开始使用Kotlin开发Android
<https://barryhappy.github.io/2016/10/20/start-kotlin-in-5-mins/>

登陆注册 Demo
<http://blog.csdn.net/xiehuimx/article/details/72354371>

Kotlin 系统入门到进阶 视频教程 <https://github.com/enbandari/Kotlin-Tutorials>


官方文档：<https://www.kancloud.cn/pholance/kotlin/125094>