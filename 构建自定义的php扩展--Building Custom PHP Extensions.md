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


接下来第一步需要编辑config.m4文件。 这个文件会被buildconf使用来更新php configure脚本. config.m4文件是一个unix m4语法文件。更多关于m4的信息可以参见http://www.gnu.org.software/m4/. 该文件包含很多注释。以dnl开头的行都是注释。

first_test 扩展生成的config.m4 文件如下：

```
dnl $Id$
dnl config.m4 for extension first_test

dnl Comments in this file start with the string 'dnl'.
dnl Remove where necessary. This file will not work
dnl without editing.

dnl If your extension references something external, use with:

dnl PHP_ARG_WITH(first_test, for first_test support,
dnl Make sure that the comment is aligned:
dnl [  --with-first_test             Include first_test support])

dnl Otherwise use enable:

dnl PHP_ARG_ENABLE(first_test, whether to enable first_test support,
dnl Make sure that the comment is aligned:
dnl [  --enable-first_test           Enable first_test support])

if test "$PHP_FIRST_TEST" != "no"; then
  dnl Write more examples of tests here...

  dnl # --with-first_test -> check with-path
  dnl SEARCH_PATH="/usr/local /usr"     # you might want to change this
  dnl SEARCH_FOR="/include/first_test.h"  # you most likely want to change this
  dnl if test -r $PHP_FIRST_TEST/$SEARCH_FOR; then # path given as parameter
  dnl   FIRST_TEST_DIR=$PHP_FIRST_TEST
  dnl else # search default path list
  dnl   AC_MSG_CHECKING([for first_test files in default path])
  dnl   for i in $SEARCH_PATH ; do
  dnl     if test -r $i/$SEARCH_FOR; then
  dnl       FIRST_TEST_DIR=$i
  dnl       AC_MSG_RESULT(found in $i)
  dnl     fi
  dnl   done
  dnl fi
  dnl
  dnl if test -z "$FIRST_TEST_DIR"; then
  dnl   AC_MSG_RESULT([not found])
  dnl   AC_MSG_ERROR([Please reinstall the first_test distribution])
  dnl fi

  dnl # --with-first_test -> add include path
  dnl PHP_ADD_INCLUDE($FIRST_TEST_DIR/include)

  dnl # --with-first_test -> check for lib and symbol presence
  dnl LIBNAME=first_test # you may want to change this
  dnl LIBSYMBOL=first_test # you most likely want to change this 

  dnl PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL,
  dnl [
  dnl   PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $FIRST_TEST_DIR/$PHP_LIBDIR, FIRST_TEST_SHARED_LIBADD)
  dnl   AC_DEFINE(HAVE_FIRST_TESTLIB,1,[ ])
  dnl ],[
  dnl   AC_MSG_ERROR([wrong first_test lib version or lib not found])
  dnl ],[
  dnl   -L$FIRST_TEST_DIR/$PHP_LIBDIR -lm
  dnl ])
  dnl
  dnl PHP_SUBST(FIRST_TEST_SHARED_LIBADD)

  PHP_NEW_EXTENSION(first_test, first_test.c, $ext_shared)
fi
```

代码片段2 执行ext_skel 脚本后生成的config.m4

需要调整16 和18行， 将行收的注释符号dnl去掉即可。 修改后的文件如下：

```
dnl $Id$
dnl config.m4 for extension first_test

dnl Comments in this file start with the string 'dnl'.
dnl Remove where necessary. This file will not work
dnl without editing.

dnl If your extension references something external, use with:

dnl PHP_ARG_WITH(first_test, for first_test support,
dnl Make sure that the comment is aligned:
dnl [  --with-first_test             Include first_test support])

dnl Otherwise use enable:

PHP_ARG_ENABLE(first_test, whether to enable first_test support,
dnl Make sure that the comment is aligned:
[  --enable-first_test           Enable first_test support])

if test "$PHP_FIRST_TEST" != "no"; then
  dnl Write more examples of tests here...

  dnl # --with-first_test -> check with-path
  dnl SEARCH_PATH="/usr/local /usr"     # you might want to change this
  dnl SEARCH_FOR="/include/first_test.h"  # you most likely want to change this
  dnl if test -r $PHP_FIRST_TEST/$SEARCH_FOR; then # path given as parameter
  dnl   FIRST_TEST_DIR=$PHP_FIRST_TEST
  dnl else # search default path list
  dnl   AC_MSG_CHECKING([for first_test files in default path])
  dnl   for i in $SEARCH_PATH ; do
  dnl     if test -r $i/$SEARCH_FOR; then
  dnl       FIRST_TEST_DIR=$i
  dnl       AC_MSG_RESULT(found in $i)
  dnl     fi
  dnl   done
  dnl fi
  dnl
  dnl if test -z "$FIRST_TEST_DIR"; then
  dnl   AC_MSG_RESULT([not found])
  dnl   AC_MSG_ERROR([Please reinstall the first_test distribution])
  dnl fi

  dnl # --with-first_test -> add include path
  dnl PHP_ADD_INCLUDE($FIRST_TEST_DIR/include)

  dnl # --with-first_test -> check for lib and symbol presence
  dnl LIBNAME=first_test # you may want to change this
  dnl LIBSYMBOL=first_test # you most likely want to change this 

  dnl PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL,
  dnl [
  dnl   PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $FIRST_TEST_DIR/$PHP_LIBDIR, FIRST_TEST_SHARED_LIBADD)
  dnl   AC_DEFINE(HAVE_FIRST_TESTLIB,1,[ ])
  dnl ],[
  dnl   AC_MSG_ERROR([wrong first_test lib version or lib not found])
  dnl ],[
  dnl   -L$FIRST_TEST_DIR/$PHP_LIBDIR -lm
  dnl ])
  dnl
  dnl PHP_SUBST(FIRST_TEST_SHARED_LIBADD)

  PHP_NEW_EXTENSION(first_test, first_test.c, $ext_shared)
fi
```

代码片段3 修改后的config.m4文件

	需要注意的是， buildconf 脚本需要新版autoconf, automake 和libtool 如果你的系统上没有装，需要下载安装后才能继续
	
接下来第二步是执行buildconf脚本。 近期版本的buildconf脚本不能直接执行一个非开发版本的php。 如果你试图执行最新版buildconf脚本， 你可能会遇到警告信息“you should not run buildconf in a release package. use buildconf -- forse to override this check”. 

如果你遇到了以上警告。 可以执行**./buildconf force** 来避免该警告。 

	
**注意！！！**	
	
	注意以上内容有些陈旧。 在新版本的php中一般无需用buildconf命令， 而是使用phpize 命令生成configure脚本(具体内容可以参见 rango 做的视频教程http://wiki.swoole.com/wiki/page/238.html )
	
接下来可以继续configure、build， 测试新的php扩展了， 执行以下命令：

```
./configure --enable-first_test
make
sudo make install 
```

顺利的话、扩展就安装成功了。 接下来需要修改php.ini文件。 在末尾增加一行**extension=first_test.so** 下面就可以测试扩展是否安装成功了
(以上会first_test扩展目录生成first_test.php 测试脚本)

执行 php first_test.php可以看到以下信息：

```
Functions available in the test extension:
confirm_first_test_compiled
calcpi
reverse
uniquechars


Congratulations! You have successfully modified ext/first_test/config.m4. Module first_test is now compiled into PHP.
```

可以看到扩展中已经包含了定义的函数。 但是这些函数还不能使用。 因为我们还没有实现它。 接下来我们要实现这几个函数


### 实现扩展

到现在为止我们还没有去检视ext_skel 脚本 生成的头文件和源码文件。 现在需要去研究他们并实现我们定义的函数。生成的头文件是一个非常标准的php扩展头文件，对于第一章我们实现的简单功能来说，无需我们修改此头文件。后面我有更详细的说明。

源码文件first_test.c, 需要我们修改，以实现预定的功能。我们打开first_test.c 找到calcpi函数，如下：

```
PHP_FUNCTION(calcpi)
{
	int argc = ZEND_NUM_ARGS();
	long iterations;
	
	if (zend_parse_parameters(argc TSRMLS_CC, "l", &iterations) == FAILURE)
		return;
	php_error(E_WARNING, "calcpi: not yet implemented");
}

```
代码段4 自动生成的calcpi函数

```PHP_FUNCTION(calcpi)```: 声明一个函数原型
接下来两行声明了一些必须局部变量 argc， iterations；最后面的```php_error```用来产生一个waning信息提示函数还没有实现。
***```zend_parse_parameters```***函数是最重要的部分， 这部分将php脚本中的变量和扩展中的c语言的变量关联起来。这部分后面会详细介绍。

接下来我们将使用一个简单的PI算法实现以上函数。以上代码变成以下：

```

PHP_FUNCTION(calcpi)
{
	int argc = ZEND_NUM_ARGS();
	long iterations;
	int index, hits;
	double randx, randy, distance, value;

	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "l", &iterations) == FAILURE) {
		return;
	}

	hits = 0;
	for (index = 0; index < iterations; index++) {
		randx = rand();
		randy = rand();

		randx /= RAND_MAX;
		randy /= RAND_MAX;

		distance = sqrt((randx * randx) + (randy * randy));

		if (distance <= 1.0) {
			hits++;
		}
		value = ((double)hits / (double)index);
		value *= 4.0;
	}

	value = ((double) hits / (double)iterations);
	value *= 4.0;
	RETVAL_DOUBLE(value);
}

```
代码5 calcpi的完整实现

上述代码去掉了waning 信息增加了pi的简单算法。对于一个扩展而言最重要的变化是最后一行***RETVAL_DOUBLE*** 表示返回值是double类型。

执行以下代码：

```
  1 <?php
  2 print(calcpi(10000));
```
得到3.1496

reverse 函数的实现如下, 以下代码演示了扩展的基本编写，并非高效c语言编程的范例。

```

PHP_FUNCTION(reverse)
{
	char *input = NULL;
	int argc = ZEND_NUM_ARGS();
	int input_len;
	char* workstr;
	int index;

	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &input, &input_len) == FAILURE) {
		return;
	}

	workstr = (char*) emalloc(input_len + 1);
	memset(workstr, 0, input_len + 1);

	for (index = 0; index < input_len; index++) {
		workstr[index] = input[input_len - (index + 1)];
	}

	RETVAL_STRING(workstr, 1);
	efree(workstr);
}

```
***代码6 reverse函数实现***

以上函数主要演示了两件事。 ***第一***，使用emalloc， efree 内存管理语法代替标准c语言中的malloc/free。 当编写扩展时，应该优先使用php内置的内存管理函数， 因为这样zend 引擎就可以管理整个内存空间。知晓哪些内存被占用。哪些内存需要自动回收，避免内存泄漏。zend引擎提供了一系列的内存管理函数、这些将会在后面介绍。

***第二*** 以上代码展示了如何返回一个字符串给php。 ```RETVAL_STRING(workstr, 1);``` 函数接受两个参数，字符串指针、bool flag（是否需要用estrdup拷贝复制该字符串）。 在上面的例子中，我分配了一段内存，用于存储逆序的字符串， 然后将该字符串通过```RETVAL_STRING```语法拷贝了一个新的字符串并返回给php。 最后，我释放了这段内存。

或者，我可以不复制，而直接将生成的字符串返回给php，但是我个人觉得最好释放到自己分配的内存，让zend管理引擎自己分配的内存。

***uniquechars函数实现如下***

```
PHP_FUNCTION(uniquechars)
{
	char *input = NULL;
	int argc = ZEND_NUM_ARGS();
	int input_len;
	zend_bool case_sensitive;
	char* workbuf;
	int index, work_index;

	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s|b", &input, &input_len, &case_sensitive) == FAILURE) {
		return;
	}

	if (argc == 1) {
		case_sensitive = 1;
	}

	work_index = 0;
	workbuf = (char*)emalloc(input_len + 1);
	memset(workbuf, 0, input_len + 1);
	for (index = 0; index < input_len; index++) {
		if (case_sensitive) {
			if (!strchar(workbuf, input[index])) {
				workbuf[work_index] = input[index];
				work_index++;
			}
		} else {
			if (!strchar(workbuf, tolower(input[index]))) {
				workbuf[work_index] = tolower(input[index]);
				work_index++;
			}
		}
	}

	array_init(return_value);
	for (index = 0; index < input_len; index++) {
		if (workbuf[index] != '\0') {
			add_next_index_stringl(return_value, &workbuf[index], 1, 1);
		}
	}

	efree(workbuf);
}

```

***代码7 uniquechars函数实现***






















