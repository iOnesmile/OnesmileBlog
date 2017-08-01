---
layout: post
title: "Gradle 入门之 Groovy 语言详解"
modified: 2017-7-31 18:54:00
excerpt: "Groovy 语法详解（类型、运算符、程序结构、闭包）..."
tags: [Gradle]
comments: true
---

Gradle 核心是基于 Groovy 脚本语言，Groovy 脚本基于 Java 且拓展了 Java。因此 Gradle 需要依赖 JDK 和 Groovy 库。

快速安装 Groovy 可以通过 Bash，命令如下：

```bash
$ curl -s get.sdkman.io | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
$ sdk install groovy
// 查看版本，判断是否成功
$ groovy -version
```
	
#### 关键字：

as、assert、break、case、catch、class、const、continue、def、default、do、else、enum、extends、false、finally、for、goto、if、implements、import、in、instanceof、interface、new、null、package、return、super、switch、this、throw、throws、trait、true、try、while


#### Hello Groovy

```groovy
#!/usr/bin/env groovy
println "Hello Groovy."
```


### 一、类型定义

#### 1，标识符

- 普通标识符：只能以字母、美元符、下划线开始，不能以数字开头。
- 引用标识符：引用标识符出现在**点后**的表达式中。

	```groovy
	// 定义一个空的 map 集合
	def map = [:]
	// 引用标示符中可以出现空格、横杆等
	map."a b-c" = "ALLOWED"
	// 断言，map 中标识符的值与右边的字符串相等
	assert map."a b-c" == "ALLOWED"
	```
		
> 注：Groovy 中所有的字符串都可以当引用标识符。


#### 2，字符串

Groovy 有 `java.lang.String` 和 `groovy.lang.GString` 两中字符串对象类型。

- **单引号字符串**   

	是 `java.lang.String` 类型，不支持站位符插值操作。
	
- **双引号字符串**
     
	是 `groovy.lang.GString` 类型，支持站位符插值操作。其中插值占位符我们可以用 `${}` 或者 `$` 来标示，`${}` 用于一般替代字串或者表达式，`$` 主要用于A.B的形式中。

- **三重单引号字符串**   

	是 `java.lang.String` 类型，不支持站位符插值操作，可以标示多行字符串。
	
- **多重双引号字符串**   

	支持站位插值操作，可以标示多行字符串。
	
- **斜线字符串**   

	和双引号字符串很类似，通常用在正则表达式中。

	```groovy
	def name = 'Test Groovy!'
	def body = 'Test $name'
	// 单引号字符串中，占位符不会被替换
	assert body == 'Test $name'
	
	// 双引号字符串，*${}* 标识，括号内面的表达式会被计算，变量会被替换
	def sum = "The sum of 2 and 3 equals ${2 + 3}"
	assert sum.toString() == 'The sum of 2 and 3 equals 5'
	
	// 双引号字符串，*$* 标识，只对 A.B 有效，对括号、闭包等无效，会抛出 groovy.lang.MissingPropertyException 异常
	def person = [name: 'Guillaume', age: 36]
	assert "$person.name is $person.age years old" == 'Guillaume is 36 years old'
	
	// 三重单引号字符串，不支持站位符插值操作
	def aMultilineString = '''line one
	line two
	line three'''
	
	// 多重双引号字符串，支持站位符插值操作
	def name = 'Groovy'
	def template = """
	    Hello, ${name}
	    Welcome.
	"""
	
	// 斜线字符串
	def fooPattern = /.*foo.*/
	assert fooPattern == '.*foo.*'
	// 多行支持
	def multilineSlashy = /one
	    two
	    three/
	// 含站位符使用支持
	def color = 'blue'
	def interpolatedSlashy = /a ${color} car/
	```
		


#### 3，字符 Characters

Groovy没有明确的Characters。但是我们可以有如下三种不同的方式来将字符串作为字符处理，譬如：   

```groovy
char c1 = 'A' 
assert c1 instanceof Character
	
def c2 = 'B' as char 
assert c2 instanceof Character
	
def c3 = (char)'C' 
assert c3 instanceof Character
```
	

#### 4，数字 Numbers

- **整型**

	和 Java 一样，支持 byte、char、short、int、long、java.lang.BigInteger

- **浮点型**

	和 Java 一样，支持 float、double、java.lang.BigDecimal
	
	```groovy
	// 整型，‘0’开头，八进制表示
	int xInt = 077
	
	// 整型，‘0x’开头，十六进制表示
	int xInt = 0x77
	
	// 整型，‘0b’开头，二进制表示
	int xInt = 0b10101111
	
	// 浮点型，科学计数表示法
	assert 1e3  ==  1_000.0
	assert 2E4  == 20_000.0
	assert 3e+1 ==     30.0
	assert 4E-2 ==      0.04
	```
		

#### 5，Booleans 类型

```groovy
def myBooleanVariable = true
boolean untypedBooleanVar = false
booleanField = true
```
	
#### 6，Lists 集合

支持 java.util.List， 可以增删改对象，列表中类型不受限制，可以用超出列表范围的数来索引列表。

```groovy
//使用动态List
def numbers = [1, 2, 3]         
assert numbers instanceof List  
assert numbers.size() == 3
	
//List中存储任意类型
def heterogeneous = [1, "a", true]
	
//判断List默认类型
def arrayList = [1, 2, 3]
assert arrayList instanceof java.util.ArrayList
	
//使用as强转类型
def linkedList = [2, 3, 4] as LinkedList    
assert linkedList instanceof java.util.LinkedList
	
//定义指定类型List
LinkedList otherLinked = [3, 4, 5]          
assert otherLinked instanceof java.util.LinkedList
	
//定义List使用
def letters = ['a', 'b', 'c', 'd']
//判断item值
assert letters[0] == 'a'     
assert letters[1] == 'b'
//负数下标则从右向左index
assert letters[-1] == 'd'    
assert letters[-2] == 'c'
//指定item赋值判断
letters[2] = 'C'             
assert letters[2] == 'C'
//给List追加item
letters << 'e'               
assert letters[ 4] == 'e'
assert letters[-1] == 'e'
//获取一段List子集
assert letters[1, 3] == ['b', 'd']         
assert letters[2..4] == ['C', 'd', 'e'] 
	
//多维List支持
def multi = [[0, 1], [2, 3]]     
assert multi[1][0] == 2 
```
	
	
#### 7，Arrays 数组

和 Java 数组类似。

```groovy
//定义初始化String数组
String[] arrStr = ['Ananas', 'Banana', 'Kiwi']  
assert arrStr instanceof String[]    
assert !(arrStr instanceof List)
	
//使用def定义初始化int数组
def numArr = [1, 2, 3] as int[]      
assert numArr instanceof int[]       
assert numArr.size() == 3
	
//声明定义多维数组指明宽度
def matrix3 = new Integer[3][3]         
assert matrix3.size() == 3
	
//声明多维数组不指定宽度
Integer[][] matrix2                     
matrix2 = [[1, 2], [3, 4]]
assert matrix2 instanceof Integer[][]
	
//数组的元素使用及赋值操作
String[] names = ['Cédric', 'Guillaume', 'Jochen', 'Paul']
assert names[0] == 'Cédric'     
names[2] = 'Blackdrag'          
assert names[2] == 'Blackdrag'
```
	

#### 8，Maps 键值对

在Groovy中键key不一定是String，可以是任何对象(实际上 Groovy 中的 Map 就是`java.util.LinkedHashMap`)。

```groovy
//定义一个Map
def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']   
//获取一些指定key的value进行判断操作
assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00'
//给指定key的对赋值value操作与判断    
colors['pink'] = '#FF00FF'           
colors.yellow  = '#FFFF00'           
assert colors.pink == '#FF00FF'
assert colors['yellow'] == '#FFFF00'
//判断Map的类型
assert colors instanceof java.util.LinkedHashMap
//访问Map中不存在的key为null
assert colors.unknown == null
	
//定义key类型为数字的Map
def numbers = [1: 'one', 2: 'two']
assert numbers[1] == 'one'
```
	

对于Map需要特别注意一种情况，如下：

```groovy
//把一个定义的变量作为Map的key，访问Map的该key是失败的
def key = 'name'
def person = [key: 'Guillaume']      
assert !person.containsKey('name')   
assert person.containsKey('key') 
	
//把一个定义的变量作为Map的key的正确写法---添加括弧，访问Map的该key是成功的
person = [(key): 'Guillaume']        
assert person.containsKey('name')    
assert !person.containsKey('key') 	
```	


### 二、运算符

下面介绍与 Java 不同的运算符，其它请参照 Java。

#### 1，次方运算符（**）

```groovy
assert  2 ** 3 == 8
	
def f = 3
f **= 2
assert f == 9
```
	
#### 2，非运算符（!）

```groovy
assert (!true)    == false    
// 支持字符串的判断，为空时返回 false，不为空时返回 true                  
assert (!'foo')   == false                      
assert (!'')      == true 
```
	
#### 3，安全占位符(?.)
	
这个运算符主要用于避免空指针异常。

```groovy
def person = Person.find { it.id == 123 }    
def name = person?.name                      
assert name == null  
```
	
#### 4，直接域访问操作符（.@）

因为Groovy自动支持属性getter方法，但有时候我们有一个自己写的特殊getter方法，当不想调用这个特殊的getter方法则可以用直接域访问操作符。

```groovy
class User {
    public final String name                 
    User(String name) { this.name = name}
    String getName() { "Name: $name" }       
}
def user = new User('Bob')
	
assert user.name == 'Name: Bob'
assert user.@name == 'Bob' 
```

	
#### 5，方法指针操作符（.&）

因为闭包可以被作为一个方法的参数，如果想让一个方法作为另一个方法的参数则可以将一个方法当成一个闭包作为另一个方法的参数。

```groovy
def list = ['a','b','c']  
//常规写法 
list.each{  
    println it  
}  

String printName(name){  
    println name  
}  

// 方法指针操作符写法，将迭代出的每一个值作为方法的参数
list.each(this.&printName)  
```
 
 
#### 6，三目运算符（?:）

```groovy
displayName = user.name ? user.name : 'Anonymous'   
// 简化为二目运算符，逻辑同上一样
displayName = user.name ?: 'Anonymous' 
```

#### 7，展开运算符（*.）

一个集合使用展开运算符可以得到一个元素为原集合各个元素执行后面指定方法所得值的集合。

```groovy
cars = [
   new Car(make: 'Peugeot', model: '508'),
   null,                                              
   new Car(make: 'Renault', model: 'Clio')]
assert cars*.make == ['Peugeot', null, 'Renault']     
assert null*.make == null 
```
	
	
### 三、程序结构

#### 1，包名

和 Java 一致。

```groovy
// defining a package named com.yoursite
package com.yoursite
```
	
#### 2，Imports 引入

常规的导包和 Java 一致，有一个特殊。

```groovy
//例1：
import groovy.xml.MarkupBuilder
	
// using the imported class to create an object
def xml = new MarkupBuilder()
assert xml != null
	
//例2：
import groovy.xml.*
	
def markupBuilder = new MarkupBuilder()
assert markupBuilder != null
assert new StreamingMarkupBuilder() != null
	
//例3：
import static Boolean.FALSE
	
assert !FALSE
	
//例4：特殊的，相当于用as取别名
import static Calendar.getInstance as now
	
assert now().class == Calendar.getInstance().class
```
	
注意：Groovy与Java类似，已经帮我们默认导入了一些常用的包，所以在我们使用这些包的类时就不用再像上面那样导入了，如下是自动导入的包列表：

```groovy
import java.lang.*
import java.util.*
import java.io.*
import java.net.*
import groovy.lang.*
import groovy.util.*
import java.math.BigInteger
import java.math.BigDecimal
```
	
#### 3，脚本与类

相对于传统的Java类，一个包含main方法的Groovy类可以如下书写：

```groovy
class Main {                                    
    static void main(String... args) {          
        println 'Groovy world!'                 
    }
}
```
	
和Java一样，程序会从这个类的main方法开始执行，这是Groovy代码的一种写法，实际上执行Groovy代码完全可以不需要类或main方法，所以更简单的写法如下：

```groovy
println 'Groovy world!'
```
	
上面这两中写法其实是一样的，具体我们可以通过如下命令进行编译为class文件：

```groovy
groovyc demo.groovy //编译Groovy源码为class
```

我们使用反编译工具可以查看到这个demo.groovy类源码如下：

```groovy
import org.codehaus.groovy.runtime.InvokerHelper
class Main extends Script {                     
    def run() {                                 
        println 'Groovy world!'                 
    }
    static void main(String[] args) {           
        InvokerHelper.runScript(Main, args)     
    }
}
```
	
可以看见，上面我们写的groovy文件编译后的class其实是Java类，该类从Script类派生而来（查阅API）；可以发现，每个脚本都会生成一个static main方法，我们执行groovy脚本的实质其实是执行的这个Java类的main方法，脚本源码里所有代码都被放到了run方法中，脚本中定义的方法（该例暂无）都会被定义在Main类中。

通过上面可以发现，Groovy的实质就是Java的class，也就是说他一定会和Java一样存在变量作用域！对哦，前面我们解释变量时竟然没说到这个东东，这里说下吧。看下面例子：

```groovy
//单个Groovy源码文件，运行会报错找不到num变量
def num = 1 
def printNum(){  
    println num  
}
	
//单个Groovy源码文件，运行会报错找不到num变量
int num = 1 
def printNum(){  
    println num  
}  
	
//单个Groovy源码文件，运行OK成功
num = 1 
def printNum(){  
    println num  
} 
```
	
上面的例子可以发现，我们如果想要在Groovy的方法中使用Groovy的变量则不能有修饰符。然而，如果我们想在B.groovy文件访问A.groovy文件的num变量咋办呢，我们可以使用Field注解，具体操作如下：

```groovy
import groovy.transform.Field;
@Field num = 1
```
	
哈哈，这就是Groovy的变量作用域了，如果你想知道上面这些写法为啥出错，很简单，自己动手整成Java源码相信你一定可以看懂为啥鸟。

### 四、闭包

1，语法

定义一个闭包：

```groovy
// [closureparameters -> ]是可选的逗号分隔的参数列表
{ [closureParameters -> ] statements }
```
	
参数可以定义，也可以不定义，如果不定义默认有一个 it 的参数。

```groovy	
//使用显示的名为参数
{ name -> println name }                            
//接受两个参数的闭包
{ String x, int y ->                                
    println "hey ${x} the value is ${y}"
}
//包含一个参数多个语句的闭包
{ reader ->                                         
    def line = reader.readLine()
    line.trim()
}
```
	
一个闭包其实就是一个groovy.lang.Closure类型的实例，因此可以如下定义：

```groovy
//定义一个Closure类型的闭包
def listener = { e -> println "Clicked on $e.source" }      
assert listener instanceof Closure
//定义直接指定为Closure类型的闭包
Closure callback = { println 'Done!' }                      
Closure<Boolean> isTextFile = {
    File it -> it.name.endsWith('.txt')                     
}
```
	
调用闭包，可以调用 call，也可以不

```groovy
def isOdd = { int i-> i%2 == 1 }                            
assert isOdd(3) == true                                     
assert isOdd.call(2) == false
```

#### 2，参数

参数有如下规则：参数类型可选，参数默认值可选，多个参数必须用逗号隔开。

```groovy
def closureWithOneArg = { str -> str.toUpperCase() }
assert closureWithOneArg('groovy') == 'GROOVY'
	
def closureWithOneArgAndExplicitType = { String str -> str.toUpperCase() }
assert closureWithOneArgAndExplicitType('groovy') == 'GROOVY'
	
def closureWithTwoArgs = { a,b -> a+b }
assert closureWithTwoArgs(1,2) == 3
	
def closureWithTwoArgsAndExplicitTypes = { int a, int b -> a+b }
assert closureWithTwoArgsAndExplicitTypes(1,2) == 3
	
def closureWithTwoArgsAndOptionalTypes = { a, int b -> a+b }
assert closureWithTwoArgsAndOptionalTypes(1,2) == 3
	
def closureWithTwoArgAndDefaultValue = { int a, int b=2 -> a+b }
assert closureWithTwoArgAndDefaultValue(1) == 3
```
	
当一个闭包没有显式定义一个参数列表时，闭包总是有一个隐式的it参数。

```groovy
def greeting = { "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
```
	
当然，如果你想声明一个不接受任何参数的闭包，且必须限定为没有参数的调用，那么你必须将它声明为一个空的参数列表，如下：

```groovy
def magicNumber = { -> 42 }
// this call will fail because the closure doesn't accept any argument
magicNumber(11)
```
	
Groovy的闭包支持最后一个参数为不定长可变长度的参数，具体用法如下：

```groovy
def concat1 = { String... args -> args.join('') }           
assert concat1('abc','def') == 'abcdef'                     
def concat2 = { String[] args -> args.join('') }            
assert concat2('abc', 'def') == 'abcdef'
	
def multiConcat = { int n, String... args ->                
    args.join('')*n
}
assert multiConcat(2, 'abc','def') == 'abcdefabcdef'
```
	

#### 3，闭包省略调用

很多方法的最后一个参数都是一个闭包，我们可以在这样的方法调运时进行略写括弧。比如：

```groovy
def debugClosure(int num, String str, Closure closure){  
      //dosomething  
}  
	
debugClosure(1, "groovy", {  
   println"hello groovy!"  
})
```
	
可以看见，当闭包作为闭包或方法的最后一个参数时我们可以将闭包从参数圆括号中提取出来接在最后，如果闭包是唯一的一个参数，则闭包或方法参数所在的圆括号也可以省略；对于有多个闭包参数的，只要是在参数声明最后的，均可以按上述方式省略。


### 四、GDK(Groovy Development Kit)

Groovy除了可以直接使用Java的JDK以外还有自己的一套GDK，其实也就是对JDK的一些类的二次封装罢了；一样，这是[GDK官方API文档](http://www.groovy-lang.org/gdk.html)，写代码中请自行查阅。

#### 1，I/O 操作

Groovy提供了很多IO操作的方法，你可以使用Java的那写IO方法，但是没有Groovy的GDK提供的简单牛逼。

```groovy
//读文件打印脚本
new File('/home/temp', 'haiku.txt').eachLine { line ->
    println line
}
	
//读文件打印及打印行号脚本
new File(baseDir, 'haiku.txt').eachLine { line, nb ->
    println "Line $nb: $line"
}
```
	
可以看见，这是一个读文件打印每行的脚本，eachLine方法是GDK中File的方法，eachLine的参数是一个闭包，这里采用了简写省略括弧。

当然了，有时候你可能更加喜欢用Reader来操作，使用Reader时即使抛出异常也会自动关闭IO。如下：

```groovy
def count = 0, MAXSIZE = 3
new File(baseDir,"haiku.txt").withReader { reader ->
    while (reader.readLine()) {
        if (++count > MAXSIZE) {
            throw new RuntimeException('Haiku should only have 3 verses')
        }
    }
}
```
	
接着我们再看几个关于读文件的操作使用，如下：

```groovy
//把读到的文件行内容全部存入List列表中
def list = new File(baseDir, 'haiku.txt').collect {it}
//把读到的文件行内容全部存入String数组列表中
def array = new File(baseDir, 'haiku.txt') as String[]
//把读到的文件内容全部转存为byte数组
byte[] contents = file.bytes
	
//把读到的文件转为InputStream，切记此方式需要手动关闭流
def is = new File(baseDir,'haiku.txt').newInputStream()
// do something ...
is.close()
	
//把读到的文件以InputStream闭包操作，此方式不需要手动关闭流
new File(baseDir,'haiku.txt').withInputStream { stream ->
    // do something ...
}
```
	
上面介绍了一些常用的文件读操作，其它的具体参见API和GDK吧。

写文件操作：

有了上面的读操作，接下来直接看几个写操作的例子得了，如下：

```groovy
//向一个文件以utf-8编码写三行文字
new File(baseDir,'haiku.txt').withWriter('utf-8') { writer ->
    writer.writeLine 'Into the ancient pond'
    writer.writeLine 'A frog jumps'
    writer.writeLine 'Water’s sound!'
}
//上面的写法可以直接替换为此写法
new File(baseDir,'haiku.txt') << '''Into the ancient pond
A frog jumps
Water’s sound!'''
//直接以byte数组形式写入文件
file.bytes = [66,22,11]
//类似上面读操作，可以使用OutputStream进行输出流操作，记得手动关闭
def os = new File(baseDir,'data.bin').newOutputStream()
// do something ...
os.close()
//类似上面读操作，可以使用OutputStream闭包进行输出流操作，不用手动关闭
new File(baseDir,'data.bin').withOutputStream { stream ->
    // do something ...
}
```
	
上面介绍了一些常用的文件写操作，其它的具体参见API和GDK吧。

文件树操作：

在脚本环境中，遍历一个文件树是很常见的需求，Groovy提供了多种方法来满足这个需求。如下：

```groovy
//遍历所有指定路径下文件名打印
dir.eachFile { file ->                      
    println file.name
}
//遍历所有指定路径下符合正则匹配的文件名打印
dir.eachFileMatch(~/.*\.txt/) { file ->     
    println file.name
}
//深度遍历打印名字
dir.eachFileRecurse { file ->                      
    println file.name
}
//深度遍历打印名字，只包含文件类型
dir.eachFileRecurse(FileType.FILES) { file ->      
    println file.name
}
//允许设置特殊标记规则的遍历操作
dir.traverse { file ->
    if (file.directory && file.name=='bin') {
        FileVisitResult.TERMINATE                   
    } else {
        println file.name
        FileVisitResult.CONTINUE                    
    }
}
```
	
执行外部程序：

Groovy提供一种简单方式来处理执行外部命令行后的输出流操作。如下：

```groovy
def process = "ls -l".execute()             
println "Found text ${process.text}"
```
	
execute方法返回一个java.lang.Process对象，支持in、out、err的信息反馈。在看一个例子，如下：

```groovy
def process = "ls -l".execute()             
process.in.eachLine { line ->               
    println line                            
}
```
	
上面使用闭包操作打印出执行命令行的输入流信息。

#### 二、有用的工具类操作

ConfigSlurper配置：

ConfigSlurper是一个配置管理文件读取工具类，类似于Java的*.properties文件，如下：

```groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()  
    app.age  = 42
    app {                  
        name = "Test${42}"
    }
''')
	
assert config.app.date instanceof Date
assert config.app.age == 42
assert config.app.name == 'Test42'
```
	
上面介绍了一些常用的属性配置操作，其它的具体参见API和GDK吧。

Expando扩展：

```groovy
def expando = new Expando()
expando.toString = { -> 'John' }
expando.say = { String s -> "John says: ${s}" }
	
assert expando as String == 'John'
assert expando.say('Hi') == 'John says: Hi'
```
	
上面介绍了一些常用的拓展操作，其它的具体参见API和GDK吧。

还有很多其他操作，这里就不一一列举，详情参考官方文档即可，譬如JSON处理、XML解析啥玩意的，自行需求摸索吧。

### 五，DSL(Domain Specific Languages)领域相关语言

这个就不特殊说明了，只在这里提一下，因为我们前边很多地方已经用过它了，加上我们只是干货基础掌握，所以不做深入探讨。

DSL是一种特定领域的语言（功能领域、业务领域），Groovy是通用的编程语言，所以不是DSL，但是Groovy却对编写全新的DSL提供了很好的支持，这些支持来自于Groovy自身语法的特性，如下：

- Groovy不需用定义CLASS类就可以直接执行脚本；
- Groovy语法省略括弧和语句结尾分号等操作；

所以说这个基础入门没必要特别深入理解，简单的前面都用过了，理解DSL作用即可，点到为止，详情参考官方文档。


> 参考链接：   
Groovy脚本基础全攻略: <http://blog.csdn.net/yanbober/article/details/49047515>   
Gradle脚本基础全攻略: <http://blog.csdn.net/yanbober/article/details/49314255>


Gradle 插件解析：

通过自定义 Gradle 插件修改编译后的 class 文件 <https://juejin.im/entry/577b03438ac2470061afb130>

在AndroidStudio中自定义Gradle插件 <http://blog.csdn.net/huachao1001/article/details/51810328>
