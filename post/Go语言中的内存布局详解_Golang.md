# Go语言中的内存布局详解_Golang

**一、go语言内存布局**

想象一下，你有一个如下的结构体。

代码如下:

```go
type MyData struct {
        aByte   byte
        aShort  int16
        anInt32 int32
        aSlice  []byte
}
```

那么这个结构体究竟是什么呢？ 从根本上说，它描述了如何在内存中布局数据。 这是什么意思？编译器又是如何展现出来呢？ 我们来看一下。 首先让我们使用反射来检查结构中的字段。

**二、反射之上**

下面是一些使用反射来找出字段大小及其偏移量（它们相对于结构的开始位于内存中的位置）的代码。 反射可以告诉我们编译器是怎么看待类型（包括结构）的。

代码如下:

```go
// First ask Go to give us some information about the MyData type
	typ := reflect.TypeOf(MyData{})
	fmt.Printf("Struct is %d bytes long\n", typ.Size())
	// We can run through the fields in the structure in order
	n := typ.NumField()
	for i := 0; i < n; i++ {
		field := typ.Field(i)
		fmt.Printf("%s at offset %v, size=%d, align=%d\n",
			field.Name, field.Offset, field.Type.Size(),
			field.Type.Align())
	}
```

除了每个字段的偏移和大小，我还打印了每个字段的对齐方式，我稍后会解释。结果如下：

```
Struct is 32 bytes long
aByte at offset 0, size=1, align=1
aShort at offset 2, size=2, align=2
anInt32 at offset 4, size=4, align=4
aSlice at offset 8, size=24, align=8
```

aByte是我们结构体中的第一个字段，偏移量为0。它使用1字节的内存。

aShort是第二个字段。它使用2字节的内存。奇怪的是偏移量是2。这是为什么呢？答案是对齐， CPU更好地访问位于2字节（“2字节边界”）的倍数的地址处的2个字节，并访问位于4字节边界上的4个字节，直到CPU的自然整数大小，在现代CPU上是8字节（64位）。

在一些较旧的RISC CPU访问错误对齐的数字引起一个故障：在一些UNIX系统上，这将是一个SIGBUS，它会停止你的程序（或内核）。一些系统能够处理这些错误并修复错误：您的代码将运行，但会缓慢的运行，因为额外的代码将由操作系统运行以修复错误。

无论如何，对齐是Go编译器跳过一个字节放置字段aShort以便它位于2字节边界的原因。因为这样，我们可以将另一个字段放进结构体中，而不使它占用更大内存。这里是我们的结构的新版本，在aByte之后立即有一个新字段anotherByte。

代码如下:

```
type MyDatab struct {
	aByte byte
	anotherByte byte
	aShort int16
	anInt32 int32
	aSlice []byte
}
```

我们再次运行反射代码，可以看到anotherByte正好在aByte和aShort之间的空闲空间。 它坐落在偏移1，aShort仍然在偏移2.现在可能是时候注意我之前提到的那个神秘对齐字段。 它告诉我们和Go编译器，这个字段需要如何对齐。
代码如下:
```
Struct is 32 bytes long
aByte at offset 0, size=1, align=1
anotherByte at offset 1, size=1, align=1
aShort at offset 2, size=2, align=2
anInt32 at offset 4, size=4, align=4
aSlice at offset 8, size=24, align=8
```

**三、看看内存**

然而我们的结构体在内存中到底是什么样子？ 让我们看看我们能不能找到答案。 首先让我们构建一个MyData实例，并填充一些值。我选择了应该容易在内存中找到的值。

代码如下:

```
data := MyData{
       aByte:   0x1,
       aShort:  0x0203,
       anInt32: 0x04050607,
       aSlice:  []byte{
              0x08, 0x09, 0x0a,
       },
 }
```

现在一些代码访问组成这个结构的字节。 我们想要获取这个结构的实例，在内存中找到它的地址，并打印出该内存中的字节。

我们使用unsafe包来帮助我们这样做。 这让我们绕过Go类型系统将指向我们的结构的指针转换为32字节数组，这个数组就是组成我们的结构体的内存数据。

代码如下:

```
dataBytes := (*[32]byte)(unsafe.Pointer(&data))
fmt.Printf("Bytes are %#v\n", dataBytes)
```

我们运行以上代码。 这是结果，第一个字段，aByte，从我们的结构中以粗体显示。 这是希望你期望的，单字节aByte = 0x01在偏移0。


代码如下:
```
Bytes are &[32]uint8{**0x1**, 0x0, 0x3, 0x2, 0x7, 0x6, 0x5, 0x4, 0x5a, 0x5, 0x1, 0x20, 0xc4, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

接下来我们来看看AShort。 这是在偏移量2的位置并且长度为2.如果你记得，`aShort = 0x0203`，但数据显示的字节是倒序。 这是因为大多数现代CPU都是Little-Endian：该值的最低位字节首先出现在内存中。

代码如下:

```
Bytes are &[32]uint8{0x1, 0x0, **0x3, 0x2**, 0x7, 0x6, 0x5, 0x4, 0x5a, 0x5, 0x1, 0x20, 0xc4, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

同样的事情发生在Int32 = 0x04050607。 最低位字节首先出现在内存中。

代码如下:

```
Bytes are &[32]uint8{0x1, 0x0, 0x3, 0x2, 0x7, 0x6, 0x5, 0x4, 0x5a, 0x5, 0x1, 0x20, 0xc4, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

**四、神秘的插曲**

现在我们看到什么？ 这是`aSlice = [] byte {0x08，0x09，0x0a} `，在偏移量8的24个字节。我没有看到我的序列0x08，0x09，0x0a的任何地方的任何符号。 这是怎么回事？

代码如下:

```
Bytes are &[32]uint8{0x1, 0x0, 0x3, 0x2, 0x7, 0x6, 0x5, 0x4, 0x5a, 0x5, 0x1, 0x20, 0xc4, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

Go反射包里自有答案。 slice在Go语言中由以下结构体表示，该结构从指针数据开始，该数据指向保存切片中的数据的存储器; 然后是该存储器中的有用数据的长度Len，以及该存储器的大小Cap。

代码如下:

```
type SliceHeader struct {
        Data uintptr
        Len  int
        Cap  int
}
```

如果把它提供给我们的代码，我们得到以下偏移和大小。 数据指针和两个长度各为8个字节，具有8个字节对齐。

代码如下:

```
Struct is 24 bytes long
Data at offset 0, size=8, align=8
Len at offset 8, size=8, align=8
Cap at offset 16, size=8, align=8
```

如果我们再看一下后面的内存结构，我们可以看到数据是在地址0x000000c42001055a。 之后，我们看到Len和Cap都是3，这是我们的数据的长度。

代码如下:

```
Bytes are &[32]uint8{0x1, 0x0, 0x3, 0x2, 0x7, 0x6, 0x5, 0x4, 0x5a, 0x5, 0x1, 0x20, 0xc4, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

我们可以直接用以下代码访问这些数据字节。 首先让我们直接访问slice头，然后打印出数据指向的内存。

代码如下:

```
dataslice := (reflect.SliceHeader)(unsafe.Pointer(&data.aSlice))
fmt.Printf("Slice data is %#v\n", 
        (*[3]byte)(unsafe.Pointer(dataslice.Data)))
```

这是输出：

```
Slice data is &[3]uint8{0x8, 0x9, 0xa}
```

**总结**

以上就是关于Go语言内存布局的全部内容了，希望本文的内容对大家学习或者使用Go语言能有所帮助，如果有疑问大家可以留言交流。
