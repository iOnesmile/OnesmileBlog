---
layout: post
title: "Logger 日志打印库详解"
modified: 2017-2-17 18:12:00
excerpt: "Logger 使用说明，日志打印规范..."
tags: [Android, View, progressbar]
comments: true
---

## Logger 日志打印库详解      

### 一、基本使用   

[Logger](https://github.com/orhanobut/logger) 是一款 Android 平台上的简单、优雅、强大的开源日志库。


#### 1，Logger 提供了以下方法：

- 打印线程的信息
- 打印类的信息
- 打印方法的信息
- 优雅的打印JSON数据
- 优雅的打印换行符
- 打印简洁的信息
- 点击日志跳转至源码

#### 2，引入依赖库，在 app 根目录的 build.gradle 文件中加入如下代码：   

	compile 'com.orhanobut:logger:1.15'

#### 3，包括的方法：  

- Logger.d("hello");   
- **Logger.d("hello %s %d", "world", 5);**   // 字符串格式化   
- Logger.d("hello");   
- Logger.e("hello");   
- Logger.w("hello");   
- Logger.v("hello");   
- Logger.wtf("hello");   
- **Logger.json(JSON_CONTENT);**   
- **Logger.xml(XML_CONTENT);**    
- Logger.log(DEBUG, "tag", "message", throwable);   

#### 4，同时支持 **List、Map、Set 和数组**的输出：

	Logger.d(list);
	Logger.d(map);
	Logger.d(set);
	Logger.d(new String[]);

#### 5，Logger 全局配置

这里是全局的 Logger 配置，如果不设置会使用默认值   

	Logger
	  .init(YOUR_TAG)                 // 默认 TAG 为 PRETTYLOGGER
	  .methodCount(3)                 // 默认方法数 2
	  .hideThreadInfo()               // 默认显示线程信息
	  .logLevel(LogLevel.NONE)        // 默认级别 LogLevel.FULL，为显示所有级别日志
	  .methodOffset(2)                // 设置调用堆栈的偏移值，默认是 0
	  .logAdapter(new AndroidLogAdapter()); // 默认 AndroidLogAdapter
	}

> LogLevel.NONE 表示不打印日志，在发布版本中用   
> 通过 logAdapter() 方法，可以自定义日志实例。需要实现 LogAdapter

#### 6，Logger 局部配置   

对单个 Log 的配置，如下：

TAG：  

	Logger.init("mytag");
	Logger.t("mytag").d("hello");

methodCount：

	Logger.init().methodCount(1);
	Logger.t(1).d("hello");


### 二、使用技巧

#### 1，默认的设置，效果如下：

<img src="http://www.ionesmile.com/images/android/logger_print_sample.png"/>

#### 2，不显示方法数和线程信息，效果如下：

	Logger.init().methodCount(0).hideThreadInfo();

<img src="http://www.ionesmile.com/images/android/logger_print_sample_only_message.png"/>

#### 3，Json 排版效果：

	Logger.json(YOUR_JSON_DATA);

<img src="http://www.ionesmile.com/images/android/logger_print_json.png"/>


### 三、日志打印规范

#### 1，日志的级别（由高到低）

- **ERROR**：系统中发生了非常严重的问题，导致业务不正常服务，必须马上进行处理。   

- **WARN**：预期会发生的，并且已经有了其他的处理流程，处理过程可以继续。

- **INFO**：重要的业务处理已经结束。如：处理机票预订的系统，对每一张票要有且只有一条INFO信息描述 "[Who] booked ticket from [Where] to [Where]"；另外显著改变应用状态的每一个 action，如数据更新，外部系统请求。   

- **DEBUG**：开发人员使用，该级别日志的主要作用是对系统每一步的运行状态进行精确的记录。通过该种日志，可以查看某一个操作每一步的执 行过程，可以准确定位是何种操作，何种参数，何种顺序导致了某种错误的发生。

- **VERBOSE**：非常具体的信息，只能限于开发调试使用，不应该编译进产品中。

#### 2，标点符号的使用   

- 表示一个进行中的过程用 ... 结尾：   
	Starting a process...

- 表示错误用 ! 结尾：   
	The name has more than 1 record!

- 正常语句用 . 结尾, 例：   
	Received an example event.


#### 3，周期性调用的信息 

有些函数可能会周期性地被调用，如果有日志的话会打印的非常多。所以要求:

- 将此类函数的正常日志信息打到debug里面
- 尽量去掉不重要信息，保留到最多一行为宜

#### 4，禁止（不适当的打印）   

- 将你认为重要但不是错误的信息打到ERROR里面

- 过于宽泛化的描述，比如:   
	log.debug("start processing...)

- 应该更明确地记录操作,比如:   
	log.debug("start retrive endpoint data processing...")

- 尽量减少使用print或者用log往终端打印信息。


　   
> 参考资料：   
> 
> <http://www.jianshu.com/p/8551fe9c6354> (LOG使用规范)   
> <https://github.com/alaudacloud/style-guides/blob/master/logs.md>   
> <http://blog.jobbole.com/56574/> (王健：最佳日志实践)   