---
title: Golang 字符串格式化
date: 2016-11-04 15:12:53 
categories: Golang 
tags: [Golang] 
description: 
---

写代码时经常会用到字符串，这里总结下 Go 语言中常用的字符串格式化例子。

<!--more-->

```go
package main

import (
	"fmt"
	"os"
)

type point struct {
	x, y int
}

func main() {
	// Go提供了几种打印格式，用来格式化一般的Go值
	// 下面的%v打印了一个结构体的对象的值
	p := point{1, 2}
	fmt.Printf("%v\n", p)

	// 如果所格式化的值是一个结构体对象，那么%+v的格式化输出将包括结构体的成员名称和值
	fmt.Printf("%+v\n", p)

	// %#v格式化输出将输出一个值的Go语法表示方式。
	fmt.Printf("%#v\n", p)

	// 使用%T来输出一个值的数据类型
	fmt.Printf("%T\n", p)

	// 使用%p输出一个指针的值
	fmt.Printf("%p\n", &p)

	// 使用%#p输出一个不带0x的指针的值
	fmt.Printf("%#p\n", &p)

	// 格式化布尔型变量
	fmt.Printf("%t\n", true)

	// 有很多的方式可以格式化整型，使用%d是一种标准的以10进制来输出整型的方式
	fmt.Printf("%d\n", 123)

	// 使用%+d可以将符号也打印出来
	fmt.Printf("%+d\n", 123)

	// 这种方式输出整型的二进制表示方式
	fmt.Printf("%b\n", 14)

	// 这种方式输出整型的不带零的八进制表示方式
	fmt.Printf("%o\n", 16)

	// 这种方式输出整型的带零的八进制表示方式
	fmt.Printf("%#o\n", 16)

	// 这里打印出该整型数值所对应的字符，即A对应的ASCII值
	fmt.Printf("%c\n", 33)

	// 输出一个值的小写的十六进制表示方式
	fmt.Printf("%x\n", 456)

	// 输出一个值的一个值的大写的十六进制表示方式
	fmt.Printf("%X\n", 456)

	// 输出带0x的十六进制
	fmt.Printf("%#x\n", 456)

	// 浮点型数值也有几种格式化方法。最基本的一种是%f
	fmt.Printf("%f\n", 78.9)

	// %e和%E使用科学计数法来输出整型
	fmt.Printf("%e\n", 123400000.0)
	fmt.Printf("%E\n", 123400000.0)

	// 使用%s输出基本的字符串
	fmt.Printf("%s\n", "\"string\"")

	// 输出像Go源码中那样带双引号的字符串，需使用%q
	fmt.Printf("%q\n", "\"string\"")

	// 输出字符串带反引号，但字符串内部不能有`字符
	fmt.Printf("%#q\n", "string")

	// %x以16进制输出字符串，每个字符串的字节用两个字符输出，大小写同前
	fmt.Printf("%x\n", "hex this")

	// 当输出数字的时候，经常需要去控制输出的宽度和精度。可以使用一个位于%后面的数字来控制输出的宽度
	// 默认情况下输出是右对齐的，左边加上空格
	fmt.Printf("|%6d|%6d|\n", 12, 345)

	// 你也可以指定浮点数的输出宽度，同时你还可以指定浮点数的输出精度
	fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)

	// 用最少的数字来表示浮点数
	fmt.Printf("%g\n", 1.23)

	// 用最多3位数字来表示浮点数
	fmt.Printf("%.3g\n", 12.34)

	// 使用 - 符号，进行左对齐
	fmt.Printf("|%-6.3f|%-6.2f|\n", 1.2, 3.45)

	// 使用 %05d 符号， 会在数字前面补零。
	fmt.Printf("%05d\n", 43)

	// 你也可以指定输出字符串的宽度来保证它们输出对齐。默认情况下，输出是右对齐的
	fmt.Printf("|%6s|%6s|\n", "foo", "b")

	// 为了使用左对齐你可以在宽度之前加上-号
	fmt.Printf("|%-6s|%-6s|\n", "foo", "b")

	// Printf函数的输出是输出到命令行os.Stdout的，你可以用Sprintf来将格式化后的字符串赋值给一个变量
	s := fmt.Sprintf("a %s", "string")
	fmt.Println(s)

	// 你也可以使用Fprintf来将格式化后的值输出到io.Writers
	fmt.Fprintf(os.Stderr, "an %s\n", "error")
}

```

运行结果

```
{1 2}
{x:1 y:2}
main.point{x:1, y:2}
main.point
0xc42007e010
c42007e010
true
123
+123
1110
20
020
!
1c8
1C8
0x1c8
78.900000
1.234000e+08
1.234000E+08
"string"
"\"string\""
`string`
6865782074686973
|    12|   345|
|  1.20|  3.45|
1.23
12.3
|1.200 |3.45  |
00043
|   foo|     b|
|foo   |b     |
a string
an error

```

