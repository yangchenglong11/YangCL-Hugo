---
title: Golang 实现大数的乘法
date: 2016-12-11 09:24:15 
categories: Golang 
tags: [Golang] 
description: 
---

利用 Go 语言实现两个大数的乘法。
<!--more-->

看到一道题目，要求模拟小学生的运算思维，实现大数的乘法。觉得不错就动手做了下，不过真正做的时候并没有完全按照小学生的思维，稍稍优化了些。

大体思路就是用字符串来表示大数，将他们存到数组中，结果也存到数组中，这样就解决了因数字太大导致计算机无法表示的问题。

可以发现，在两个数进行运算时，积最多为 m+n 位，而且第一个乘数的第 m 位与第二个乘数的第 n 位的结果应在结果的第 m+n 位。有了思路。就开始编码吧！下面附上代码。

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
	"strings"
)

func mutl(a, b string) {
	num_a := make([]int, len(a)-1)
	str_a := strings.Split(a, "")
	for i := 0; i < len(str_a)-1; i++ {
		num_a[i], _ = strconv.Atoi(str_a[i])
	}

	num_b := make([]int, len(b)-1)
	str_b := strings.Split(b, "")
	for i := 0; i < len(str_b)-1; i++ {
		num_b[i], _ = strconv.Atoi(str_b[i])
	}

	total := len(num_a) + len(num_b)
	result := make([]int, total)

	var (
		index1   = 0
		index2   = 0
		resIndex = 0
		segRes   = 0
	)

	for i := len(num_a); i > 0; i-- {
		index1 = len(num_a) - i

		for j := len(num_b); j > 0; j-- {
			index2 = len(num_b) - j

			resIndex = index1 + index2
			segRes = num_a[index1] * num_b[index2]

			carry(&result, resIndex, segRes)
		}
	}

	fmt.Print("结果为：")
	for i := 0; i < total-1; i++ {
		fmt.Print(result[i])
	}
	fmt.Println()
}

func carry(arr *[]int, index int, value int) {
	(*arr)[index] += value

	if (*arr)[index] > 9 {
		result := (*arr)[index] / 10
		(*arr)[index] = (*arr)[index] % 10
		carry(arr, index-1, result)
	}
}

func main() {
	aa := bufio.NewReader(os.Stdin)
	fmt.Print("请输入一个乘数: ")
	a, _ := aa.ReadString('\n')

	bb := bufio.NewReader(os.Stdin)
	fmt.Print("请输入另一个乘数: ")
	b, _ := bb.ReadString('\n')

	mutl(a, b)
}
```