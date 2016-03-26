# 构建自定义的php扩展--Building Custom PHP Extensions

## 翻译说明
本文是Building Custom PHP Extensions 书籍的中文翻译版， 是一本关于扩展开发的手册。 本书并没有大篇幅讲解php的内核结构， 而是着重讲解扩展开发的步骤和常用工具函数。 通过本书可以对扩展开发快速入门。 

翻译时忠实原文、但不逐字逐句、译本假定读者有一定php基础并且对php扩展以及c语言有基本了解。剔除了部分冗余的内容。想看原版的读者可以自行购买原版，书比较薄、很便宜(网上没有电子书~~)。 该书翻译完全利用周末时间进行、所以进度不会很快 -_-#

## 关于译者

1. 邮箱：fanhengguang@126.com
2. qq: 459173139

## 第一个PHP扩展
### 构建脚本
下载php源码， 然后进入源码中的扩展目录：


	cd /Users/fanhenguang/work/c_learn/php_5.x.x/ext
	
此目录中你会看到***ext_skel*** 命令。 执行该命令不带任何参数。你将会看到该命令的帮助说明

	./ext_skel --extname=module [--proto=file] [--stubs=file] [--xml[=file]]
	           [--skel=dir] [--full-xml] [--no-help]
	
	  --extname=module   module is the name of your extension
	  --proto=file       file contains prototypes of functions to create
	  --stubs=file       generate only function stubs in file
	  --xml              generate xml documentation to be added to phpdoc-cvs
	  --skel=dir         path to the skeleton directory
	  --full-xml         generate xml documentation for a self-contained extension
	                     (not yet implemented)
	  --no-help          don't try to be nice and create comments in the code
	                     and helper functions to test if the module compiled 
	                     
***参数解析如下：***


**表1：ext_skel 脚本参数说明**


参数  | 说明
------------- | -------------
--extname=module   | 指定扩展名称
--proto=file  | 用于指定函数原型， php会根据原型自动生成函数声明
 --stubs=file  |  which will leave out also all module specific stuff and write just function stubs with function value declarations and passed argument handling, and function entries and definitions at the end of the file, for copying and pasting into an already existing module.
--xml|生成xml文档， 将会被添加到phpdoc-cvs 中
--skel=dir|设定框架目录
--full-xml|预留功能、暂未实现
--no-help| 默认情况，ext_skel 将会生成很多注释和一个测试函数。用于帮助初学者更好理解扩展结构。 执行脚本是加上该参数，将会关闭这些注释。因为这些注释对于有经验的开发者来说是毫无意义的。这个参数长和--stubs=file 一起使用。 


	注：建议读者阅读README.EXT_SKEL 文档（php 源码目录中）获取更多关于ext_skel 的详细说明

对于我们的第一个扩展，我们将实现以下几个简单函数：

```php
doubel calcpi(int iterations)
```

这个函数用于计算π值。所用算法是一个简单的随机数字方法。计算结果并不精确。但是会比较有趣

```php
string reverse(string input)
```

该函数将会返回一个字符串的翻转版本。 "Test String" => "gnirtS tseT";


```php
array uniquechars(string input, [bool case_sensitive])
```
此函数返回一个字符串的非重复字符构成的数组， 例如：“Test String” 返回一个包含“T”, 'e', 's', 't', 'S', 'r', 'i' , 'n', 'g' 构成的数组。 case_sensitive 是一个可选参数
默认为true表示字符是大小写敏感的。 如果case\_sensitive=false 上面的例子将会返回't', 'e', 's', 'r', 'i', 'n', 'g' 构成的数组。

以上函数虽然在实际应用中用处不大， 但是作为扩展的例子，这些例子可以涉及到扩展开发的很多特性。而这些对于一个复杂扩展是必不可少的。 

首先编写原型文件， 格式为每个原型函数占一行，对于上面提到的三个函数，原型文件如下：


***代码1 first_test 扩展原型文件***

	```
	double calcpi( int iterations) Calculate Pi
	string reverse( string input) reverse the input string
	array uniquechars( string input [, bool case_sensitive]) return the unique characters in the input string
	
	```

以上代码段命名为first_test.proto. ext_skel脚本将会读取处理该文件，生成php扩展所需的c语言框架。需要指出的是，原型文件并非必须， 但是利用原型文件可以减少c语言编写工作量，并且减少错误的产生。（对于有经验的开发者。原型文件是非必须的）。

以上代码，每一行代表一个函数，此函数将会用c语言实现，并提供给php进行调用。原型的格式如下：


```
[function return type] function name (parameter list) option comment
```

**function return type** 即函数返回类型是非必须的。 并不会对生成的扩展框架产生硬性。仅仅对内核文档起作用。**parameter list** ：逗号分隔的参数列表，eg， string param1, string param2. [] 括起来的参数表示可选参数。

有了以上原型文件，下面可以执行ext_skel脚本了：

```
./ext_skel --extname=first_test --proto=first_test.proto
```

假设你有写权限、以上命令将会执行并且输出一个简介，提示你编译、安装一个新扩展 如下：

	1.  $ cd ..
	2.  $ vi ext/first_test/config.m4
	3.  $ ./buildconf
	4.  $ ./configure --[with|enable]-first_test
	5.  $ make
	6.  $ ./sapi/cli/php -f ext/first_test/first_test.php
	7.  $ vi ext/first_test/first_test.c
	8.  $ make
	
	

按照以上步骤即可正确的编译安装php扩展。在编译安装扩展之前、检视一下php/ext/first_test
目录下生成的文件是很有趣的。文件如下：

文件名  | 含义
------------- | -------------
config.m4 | 这个文件将会被buildconf 脚本使用，用于将新扩展集成到php configure 脚本中
CREDITS  | 一个空文件，用于极少扩展作者的信息
EXPERIMENTAL  | 空文件
first_test.c  | 新扩展的逻辑实现
first_test.php  | 用于测试扩展的php脚本
php_first_test.h  | 头文件
tests  | php测试脚本目录
***表2 ext_skel脚本生成的文件***
















