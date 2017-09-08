---
layout: post
title: "Kotlin 官方文档学习笔记 - 概述"
modified: 2017-9-8 12:35:00
excerpt: "Kotlin 新特性、学习文档、测试工具..."
tags: [Kotlin]
comments: true
---

- Kotlin 用于服务器端
	
	语言什么都很厉害，完全可以代替 Java。有服务端的开发框架，如 Spring。可以部署在支持 JavaWeb 的任何主机。

- Kotlin 用于 Android

	兼容 JDK6、性能与 Java 类似、与 Java 100% 互操作性、额外占用小、增量编译与 Java 类似。

- Kotlin 用于 JavaScript

	支持 ECMAScript 5.1 标准，代码会转成 JavaScript。可以与 JQuery、ReactJS 一起使用。

- #### 新特性

	1. 协程(??)

		如在 UI 线程中异步启动一个网络线程，当网络线程完成后回来更新 UI。代码如下：
		
		```kotlin
		// 在 UI 上下文中启动新的协程
		launch(UI) {
		    // 等待异步叠加完成
		    val image = asyncOverlay().await()
		    // 然后在 UI 中显示
		    showImage(image)
		}
		```
		
		 `yield` 和 `yieldAll` 函数使用协程来支持惰性生成序列?
		
		
	2. 类型别名

		类型别名允许你为现有类型定义备用名称。
		
		```kotlin
		typealias OscarWinners = Map<String, String>
		```
		
	
	3. 已绑定的可调用引用(?)
	
		现在可以使用 `::` 操作符来获取指向特定对象实例的方法或属性的成员引用
	
		```kotlin
		val numberRegex = "\\d+".toRegex()
		val numbers = listOf("abc", "123", "456").filter(numberRegex::matches)
		```
	
	
	4. 密封类和数据类(???)

		数据类现在可以扩展其他类。 这可以用来友好且清晰地定义一个表达式类的层次结构：

		```kotlin
		sealed class Expr

		data class Const(val number: Double) : Expr()
		data class Sum(val e1: Expr, val e2: Expr) : Expr()
		object NotANumber : Expr()
		
		fun eval(expr: Expr): Double = when (expr) {
		    is Const -> expr.number
		    is Sum -> eval(expr.e1) + eval(expr.e2)
		    NotANumber -> Double.NaN
		}
		val e = eval(Sum(Const(1.0), Const(2.0)))
		```
	
		
	5. lambda 表达式中的解构

		可以使用解构声明语法来解开传递给 lambda 表达式的参数。
						
		```kotlin
		val map = mapOf(1 to "one", 2 to "two")
		// 之前
		println(map.mapValues { entry ->
		                       val (key, value) = entry
		                       "$key -> $value!"
		                      })
		// 现在
		println(map.mapValues { (key, value) -> "$key -> $value!" })
		```
		
		
	6. 下划线用于未使用的参数

		对于具有多个参数的 lambda 表达式，可以使用 _ 字符替换不使用的参数的名称
				
		```kotlin
		map.forEach { _, value -> println("$value!") }
		// 这也适用于解构声明
		val (_, status) = getResult()
		```	
		
		
	7. 数字字面值中的下划线

		正如在 Java 8 中一样，Kotlin 现在允许在数字字面值中使用下划线来分隔数字分组
		
		```kotlin
		val oneMillion = 1_000_000
		val hexBytes = 0xFF_EC_DE_5E
		val bytes = 0b11010010_01101001_10010100_10010010
		```
		
	
	8. 对于属性的更短语法
		
		对于没有自定义访问器、或者将 getter 定义为表达式主体的属性，现在可以省略属性的类型
		
		```kotlin
		data class Person(val name: String, val age: Int) {
		    val isAdult get() = age >= 20 // 属性类型推断为 “Boolean”
		}
		```
		
	9. 内联属性访问器

		如果属性没有幕后字段，现在可以使用 inline 修饰符来标记该属性访问器。 这些访问器的编译方式与内联函数相同。
		
		```kotlin
		public val <T> List<T>.lastIndex: Int
		inline get() = this.size - 1
		```
		
	10. 局部委托属性(?)

		现在可以对局部变量使用委托属性语法。 一个可能的用途是定义一个延迟求值的局部变量
		
		```kotlin
		val answer by lazy {
		    println("Calculating the answer...")
		    42
		}
		if (needAnswer()) {                     // 返回随机值
		    println("The answer is $answer.")   // 此时计算出答案
		}
		else {
		    println("Sometimes no answer is the answer...")
		}
		```
		
		
	11. 委托属性绑定的拦截(???)

		对于委托属性，现在可以使用 provideDelegate 操作符拦截委托到属性之间的绑定 。provideDelegate 方法在创建 MyUI 实例期间将会为每个属性调用，并且可以立即执行必要的验证。
		
		```kotlin
		class ResourceLoader<T>(id: ResourceID<T>) {
		    operator fun provideDelegate(thisRef: MyUI, property: KProperty<*>): ReadOnlyProperty<MyUI, T> {
		        checkProperty(thisRef, property.name)
		        …… // 属性创建
		    }
		
		    private fun checkProperty(thisRef: MyUI, name: String) { …… }
		}
		
		fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { …… }
		
		class MyUI {
		    val image by bindResource(ResourceID.image_id)
		    val text by bindResource(ResourceID.text_id)
		}
		```
		
		
	12. 泛型枚举值访问(??)

		现在可以用泛型的方式来对枚举类的值进行枚举
		
		```kotlin
		enum class RGB { RED, GREEN, BLUE }
		
		inline fun <reified T : Enum<T>> printAllValues() {
		    print(enumValues<T>().joinToString { it.name })
		}
		```
		
		
	13. 对于 DSL 中隐式接收者的作用域控制(???)   ->>> gradle 表达式？

		@DslMarker 注解允许限制来自 DSL 上下文中的外部作用域的接收者的使用。
		
		```kotlin
		table {
		    tr {
		        td { +"Text" }
		    }
		}
		```
		
		
	14. rem 操作符

		mod 操作符现已弃用，而使用 rem 取代
		
		
	15. 标准库
		
		- 字符串到数字的转换

		- onEach()

		- also()、takeIf() 和 takeUnless()

		- groupingBy()

		- Map.toMap() 和 Map.toMutableMap()

		- Map.minus(key)

		- minOf() 和 maxOf()

		- 类似数组的列表实例化函数

		- Map.getValue()

		- 抽象集合

		- 数组处理函数
		