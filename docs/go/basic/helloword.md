## Go语言初步了解

### Go语言结构

Go 语言的基础组成有以下几个部分：

- 包声明
- 引入包
- 函数
- 变量
- 语句 & 表达式
- 注释

示例代码：

```
package main

import "fmt"

func main() {
   /* 这是我的第一个简单的程序 */
   fmt.Println("Hello, World!")
}

```

输出结果：

```
Hello, World!

```



1、第一行代码 *package main* 定义了包名。你必须在源文件中非注释的第一行指明这个文件属于哪个包，如：package main。package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。

2、下一行 *import "fmt"* 告诉 Go 编译器这个程序需要使用 fmt 包（的函数，或其他元素），fmt 包实现了格式化 IO（输入/输出）的函数。

3、下一行 *func main()* 是程序开始执行的函数。main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行该函数）。

4、下一行 /*...*/ 是注释，在程序执行时将被忽略。单行注释是最常见的注释形式，你可以在任何地方使用以 // 开头的单行注释。多行注释也叫块注释，均已以 /* 开头，并以 */ 结尾，且不可以嵌套使用，多行注释一般用于包的文档描述或注释成块的代码片段。

5、下一行 *fmt.Println(...)* 可以将字符串输出到控制台，并在最后自动增加换行字符 \n。
使用 fmt.Print("hello, world\n") 可以得到相同的结果。
Print 和 Println 这两个函数也支持使用变量，如：fmt.Println(arr)。如果没有特别指定，它们会以默认的打印格式将变量 arr 输出到控制台。

6、当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 protected ）。

注意：

需要注意的是 **{** 不能单独放在一行，所以以下代码在运行时会产生错误：

```
package main

import "fmt"

func main()  
{  // 错误，{ 不能在单独的行上
    fmt.Println("Hello, World!")
}
```

go语法：

注释：

以 // 开头的单行注释。多行注释也叫块注释，均已以 /* 开头，并以 */ 结尾。如：

```
// 单行注释
/*
 Author by 菜鸟教程
 我是多行注释
 */
```

字符串连接符号:

Go 语言的字符串可以通过 **+** 实现

```
package main
import "fmt"
func main() {
    fmt.Println("Google" + "Runoob")
}
```

```
# 输出结果
GoogleRunoob
```

----



### 关键字

下面列举了 Go 代码中会使用到的 25 个关键字或保留字：

| break    | default     | func   | interface | select |
| -------- | ----------- | ------ | --------- | ------ |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

除了以上介绍的这些关键字，Go 语言还有 36 个预定义标识符：

| append | bool    | byte    | cap     | close  | complex | complex64 | complex128 | uint16  |
| ------ | ------- | ------- | ------- | ------ | ------- | --------- | ---------- | ------- |
| copy   | false   | float32 | float64 | imag   | int     | int8      | int16      | uint32  |
| int32  | int64   | iota    | len     | make   | new     | nil       | panic      | uint64  |
| print  | println | real    | recover | string | true    | uint      | uint8      | uintptr |

程序一般由关键字、常量、变量、运算符、类型和函数组成。

程序中可能会使用到这些分隔符：括号 ()，中括号 [] 和大括号 {}。

程序中可能会使用到这些标点符号：.、,、;、: 和 …。

---





### Go语言数据类型

| 序号 | 类型和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **布尔型** 布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true。 |
| 2    | **数字类型** 整型 int 和浮点型 float32、float64，Go 语言支持整型和浮点型数字，并且支持复数，其中位的运算采用补码。 |
| 3    | **字符串类型:** 字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本。 |
| 4    | **派生类型:** 包括：(a) 指针类型（Pointer）(b) 数组类型(c) 结构化类型(struct)(d) Channel 类型(e) 函数类型(f) 切片类型(g) 接口类型（interface）(h) Map 类型 |

数字类型

Go 也有基于架构的类型，例如：int、uint 和 uintptr。

#### 整数类型

| 序号 | 类型和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **uint8** 无符号 8 位整型 (0 到 255)                         |
| 2    | **uint16** 无符号 16 位整型 (0 到 65535)                     |
| 3    | **uint32** 无符号 32 位整型 (0 到 4294967295)                |
| 4    | **uint64** 无符号 64 位整型 (0 到 18446744073709551615)      |
| 5    | **int8** 有符号 8 位整型 (-128 到 127)                       |
| 6    | **int16** 有符号 16 位整型 (-32768 到 32767)                 |
| 7    | **int32** 有符号 32 位整型 (-2147483648 到 2147483647)       |
| 8    | **int64** 有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |

示例1：int8溢出，使用时要注意数的范围

```
package main

import "fmt"

func main()  {
	var num int8 = 128

	fmt.Println(num)
}
```

结果

```
[root@k8s2 go-code]# go run "/usr/local/go/src/github.com/serialt/go-code/0206/test2/main.go"
# command-line-arguments
0206/test2/main.go:6:6: constant 128 overflows int8
[root@k8s2 go-code]# 
```

整数使用细节

* Golangd的个整数类型分：有符号和无符号，int uint的大小与系统有关，32位系统就是32位，64位系统就是64位。
* Golang的整数默认声明为int型
* 如何在程序中查看某个变量的字节大小和数据类型
* Golang程序中整型变量在使用时，遵守保小不保大的原则。即在保证程序正常运行的前提下，尽量使用占用内存小的数据类型。【如：年龄】
* bit：计算机中最小的存储单位。byte：计算机中基本的存储单元。1 byte = 8 bit

```
package main

import "fmt"
import "unsafe"

func main()  {
	var num =100

	//判断变量的类型
	fmt.Printf("num的数据类型是 %T",num)

	var num2 int64 = 10

	fmt.Printf("\nnum2的数据类型是：%T\n占用的字节数是: %d",num2  ,unsafe.Sizeof(num2))
}
```

```
[root@k8s2 go-code]# go run "/usr/local/go/src/github.com/serialt/go-code/0206/test2/main.go"
num的数据类型是 int
num2的数据类型是：int64
占用的字节数是: 8
```



#### 浮点型

| 序号 | 类型和描述                        |
| :--- | :-------------------------------- |
| 1    | **float32** IEEE-754 32位浮点型数 |
| 2    | **float64** IEEE-754 64位浮点型数 |
| 3    | **complex64** 32 位实数和虚数     |
| 4    | **complex128** 64 位实数和虚数    |

------

```
package main

import (
	"fmt"
)

func main(){
	var price float32 = 99.99
	fmt.Println(price)

}
```

浮点型使用细节

* Golang浮点类型有固定的长度，不受具体OS的影响
* Golang的浮点类型默认声明为float64
* 通常情况下应该使用float64，因为他比float32更精确。



#### 字符类型：

Golang中没有专门用来保存字符类型，如果要村单个字符（字母），一般用`byte`来保存。

字符串就是一串固定长度的字符连接起来的字符序列，Go的字符串是有单个字节连接起来的。Go发字符串是由字节组成的。

使用注意：

* 字符常量使用单引号('')括起来的单个字符，例如：var c1 byte = 'a' 。
* Go中允许使用转义字符`'\'`将其后的字符转变为特俗字符型常量。
* Go语言的字符使用`UTF-8`编码

* 在Go中，字符的本质是一个整数，直接输出时，是该字符的对应的UTF-8编码的码值。
* 可以直接给某个变量赋予一个数字，然后按格式化输出时`%c`，会输出该数字对应的unicode字符
* 字符类型时可以进行运算的，相当于一个整数，因为它都有对应的Unicode码

其他数字类型

以下列出了其他更多的数字类型：

| 序号 | 类型和描述                               |
| :--- | :--------------------------------------- |
| 1    | **byte** 类似 uint8                      |
| 2    | **rune** 类似 int32                      |
| 3    | **uint** 32 或 64 位                     |
| 4    | **int** 与 uint 一样大小                 |
| 5    | **uintptr** 无符号整型，用于存放一个指针 |



#### 布尔类型

| 序号 | 类型和描述 |
| ---- | ---------- |
| 1    | true 真    |
|      | false 假   |

说明：

* bool类型是允许取值true和false。
* bool类型只占一个字节
* boolean类型适用与逻辑运算，一般用于程序的流程控制。if判断语句和for循环语句。



#### 字符串类型

使用细节：

* Go语言的字符串适用UTF-8识别Unicode文本，这样Golang统一适用UTF-8编码，编码问题不会再困扰程序员。
* 字符串一旦赋值，字符串就不能修改了，再Go中字符串时不可改变的。
* 字符串的表示形式有两种。双引号：会识别转移字符。反引号：以字符串的原生形式输出，包括换行和特殊字符。

* 字符串的拼接方式：使用`+`



基本数据类型的默认值

| 数据类型 | 默认值 |
| -------- | ------ |
| 整型     | 0      |
| 浮点型   | 0      |
| 字符串   | ""     |
| 布尔类型 | false  |
|          |        |

基本数据类型转换

Go和java和c不同，Go再不同类型的变量之间赋值需要显示转换。也就是说Go数据类型不能自动转换。

```
package main

import "fmt"

func main (){
	var i int32 = 100
	var n1 float32 = float32(i)
	var n2 int32 = int32(n2)
	fmt.Print("i=%v,n1=%v",i,n1)
}
```

基本数据类型和string间的转换

基本数据类型

方式1）fmt.Sprintf("%参数"，表达式)

a、参数需要和表达式的数据类型相匹配

b、fmt.Sprintf()会返回转换后的字符串

方式2）使用strconv包的函数

使用实例：

```
package main

import (
	"fmt"
	"strconv"
)

func main(){

	//基本数据类型转string
	var num1 int = 99
	var num2 float64 = 88.88
	var b bool 
	var mychar byte = 's'
	var str string

	//使用第一种方式转换，fmt.Sprintf
	fmt.Println("CentOS")
	str = fmt.Sprintf("%d",num1)
	fmt.Printf("str type is %T str=%v \n",str,str)

	str = fmt.Sprintf("%f",num2)
	fmt.Printf("str type is %T str=%v \n",str,str)

	str = fmt.Sprintf("%t",b)
	fmt.Printf("str type is %T str=%v \n",str,str)

	str = fmt.Sprintf("%c",mychar)
	fmt.Printf("str type is %T str=%v \n",str,str)


	//第二种方式：strconv
	str = strconv.FormatInt(int64(num1),10)
	fmt.Printf("str type is %T str=%v \n",str,str)
	
	str = strconv.FormatFloat(float64(num1),'f',10,64)	
	fmt.Printf("str type is %T str=%v \n",str,str)

}
```

输出结果

```
CentOS
str type is string str=99 
str type is string str=88.880000 
str type is string str=false 
str type is string str=s 
str type is string str=99 
str type is string str=99.0000000000 
```





































