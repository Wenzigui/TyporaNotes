[toc]

>   入门教程推荐: https://github.com/Unknwon/the-way-to-go_ZH_CN

# 概述

Go 语言的主要目标是将静态语言的安全性和高效性与动态语言的易开发性进行有机结合

类型安全，内存安全，虽然存在指针，但是不允许进行指针运算

Go 语言没有类和继承的概念，通过接口实现多态



## 特性缺失

-   不支持函数重载，操作符重载
-   不支持类型隐式转换
-   放弃类和继承，通过另一种途径来实现面向对象设计
-   不支持泛型
-   通过panic - recover替代try - catch
-   不支持静态变量



## 环境变量

-   $GOROOT：表示Go在电脑上安装的位置

-   $GOARCH：表示机器的处理器架构

-   $GOOS：表示机器的操作系统

-   $GOBIN：表示表一起和链接器的安装位置，默认是$GOROOT/bin

-   $GOPATH：默认值和$GOROOT一样，但是从Go1.1版本开始，必须修改为其它路径。该路径下必须分别包含三个规定的目录：src、pkg和bin，这三个目录分别用于存放源码文件，包文件，和可执行文件



## go runtime

>   类似于JVM

职责：

-   内存分配，垃圾回收，反射等

由C语言编写，存放在$GOROOT/src/runtime目录



# 基础

## 文件名
- 小写字母组成，多个小写字母通过下划线_分隔
- 不包含空格或其他特殊字符



## 标识符
- 无效标识符
    - 数字开头
    - Go的关键字
    - 运算符
- 区分大小写
- 空白标识符_
    1. 可用于变量声明或赋值
    2. 任何赋值给这个标识符的值都将被抛弃
- 匿名变量
- 关键字
![image-20200716154519496](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716154519496.png)
- 预定义标识符
![image-20200716154547930](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716154547930.png)
- 大小写可见性规则
    - 大写字母开头，函数/变量可以被外部包调用，遵循Pascal命名法
    - 小写字母开头，函数/变量只能在内部包调用，遵循驼峰命名法



## 包

- 每个程序都由包组成
- package main表示一个可独立执行的程序
- 包使用小写字母命名
- 是编译的最小单位
> 如果对一个包进行更改或重新编译，所有引用这个包的客户端都必须全部重新编译
- 通过import关键字导入包



## 标准库
> Go安装目录下可直接使用的包
- 存放在\$GOROOT/pkg/\$GOOS_$GOARCH/目录下



## 函数
- func funcName()



## 注释
> 跟Java一样



## 类型转化
- 必须显式说明，Go不存在隐式的类型转化
```Go
valueOfTypeB = typeB(valueOfTypeA)
```



## 常量

- 通过const关键字定义，用于存储不会改变的数据
```Go
const identifier [type] = value
```
- 可用于枚举类型
```go
const (
	Unknown = 0
	Female = 1
	Male = 2
)
```
- iota
```Go
const (
	a = iota
	b = iota
	c = iota
)
```
> 第一个iota等于0，每当iota在新的一行被使用时，它的值都会自动加1，每次遇到const关键字，都会重置为0



## 变量

- 变量声明形式
```go
var identifier type
```

- 声明包级别的全局变量
```go
var (
	HOME = os.Getenv("HOME")
	USER = os.Getenv("USER")
	GOROOT = os.Getenv("GOROOT")
)
```

- 在函数体内声明局部变量
```go
a := 1
```

- 变量命名规则 - 驼峰命名法
- Go编译器可以在编译时根据变量的值，自动推断其类型
- 值类型，引用类型
- init函数初始化变量
    - 不能手动调用，在每个包完成初始化后自动执行，执行优先级高于main函数
    - 每个源文件只能包含一个init函数
    - init函数单线程执行，按照包的依赖关系顺序执行
```go
func init() {
   // init() function
}
```



## 数据类型

### 布尔类型bool

- true
- false



### 数字类型

- 整形int
- 浮点型float

> 尽量使用float64，因为math包中所有有关数学类型的函数都会要求接收这种类型的参数



### 复数类型

- 32位实数和虚数
- 64位实数和虚数



## 格式化说明符

> 用于格式化字符串

- %d 表示格式化整数
- %x 表示16进制数
- %g 表示格式化浮点数
    - %f 输入浮点数
    - %e 输出科学计数法
- %v 表示复位
- %b 表示位



## 结构控制

>   if else太简单了，就写了

### switch

```go
switch var1 {
	case val1:
		...
	case val2:
		...
	default:
		...
}
```

- 接收任意形式的表达式
- fallthrough关键字

> 表示执行完一个分支的代码后，继续执行后续分支的代码



### for

```go
for i := 0; i < 5; i++ {
		fmt.Printf("This is the %d iteration\n", i)
}
```

- 循环头部使用分号分隔，不需要()将头部括起来
- for - range结构
    - 可以迭代任何一个集合，包括数组和Map
    - val为集合中对应索引的值拷贝，对val做任何修改都不会影响到集合中原有的值，除非val是指针

> 类似于Java中的增强for循环

```go
for ix, val := range coll { }
```



# 函数

> Go不允许函数重载

## 函数类型

- 普通带名字的函数
- 匿名函数/lambda函数
- 方法



## 函数签名

- 指函数参数、返回值以及它们的类型
- 函数的返回值可以有多个



## defer关键字

- 允许推迟到函数返回之前（或任意位置执行return语句之后）的一刻才执行某个语句或函数

> 即在return之后，返回之前执行某个语句或函数



## 变长参数

- 函数的最后一个入参为`...type`的形式
- 变长参数可以通过for - range结构获取

```go
func min(s ...int) int {
	if len(s)==0 {
		return 0
	}
	// 实际上就是一个变长数组
	min := s[0]
	// 可以使用for - range结构获取数据
	for _, v := range s {
		if v < min {
			min = v
		}
	}
	return min
}
```



## 匿名函数

- 不能独立存在，但是可以被赋值给某个变量，然后通过变量名调用

```go
fplus := func(x, y int) int { return x + y }
// 通过变量名调用函数
fplus(3,4)
```

- 可以直接调用

```go
func(x, y int) int { return x + y } (3, 4)
// 匿名函数后面直接加上括号即可直接调用
```



# 数组

> 数组是具有相同类型的一组长度固定的数据序列，如果想让数组中的元素为任意类型，可以使用空接口作为数组的类型



## 声明方式

```go
var identifier [len]type
```



## 初始化方式

### for循环初始化数组

```go
var arr1 [5]int

// for循环初始化数组
for i:=0; i < len(arr1); i++ {
    arr1[i] = i * 2
}
```



### new函数创建数组

```go
var arr1 = new([5]int)
```

> 通过new函数创建数组，返回的是指针类型的数组变量 *[5]int



### 数组常量方式创建数组

```go
var arr1 = [5]int{18, 20, 15, 22, 16}
var arr2 = [...]int{5, 6, 7, 8, 22}
var arr3 = []int{5, 6, 7, 8, 22}	//注：初始化得到的实际上是切片slice
var arr4 = [5]string{3: "Chris", 4: "Ron"}
```



==大数组在函数中传递会消耗很多内存，可以通过传递数组指针或者使用数组切片来避免==



# 切片

切片是对数组一个连续片段的引用（切片是一个引用类型）



## 声明&初始化方式

```go
// 切片声明
var identifier []type
// 初始化
// 表示 slice1 是由数组 arr1 从 start 索引到 end-1 索引之间的元素构成的子集
var slice1 []type = arr1[start:end]
```



## 内存结构

> 切片实际上是一个结构体，用三个字段：指向相关数组的指针，切片长度，切片容量

![image-20200716160624662](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716160624662.png)



## make()函数创建切片

> 如果切片相关的数组还没定义，可以先用make()函数创建切片

```go
var slice1 []type = make([]type, len)
slice2 := make([]type, len)
slice3 := make([]type, len, cap)
```

make方法生成的切片的内存结构

![image-20200716160821465](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716160821465.png)

> **make函数和new函数都可以生成切片**
>
> - make([]int, 2, 5)
> - new([5]int)[2:5]



## 【重要】new()跟make()的区别

- new()为每个类型分配一片内存，初始化为0并且返回类型为\*T的内存地址，** 适用于数组、结构体 **

> 返回一个指向类型为T，值为0的地址的指针，相当于&T{}

- make()返回一个类型为T的初始值，**仅适用于三种内建的引用类型：切片、map和channel**

![image-20200716162412179](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716162412179.png)

## 切片重组

> 改变切片长度的过程称为切片重组（reslicing）

```go
    var ar = [10]int{0,1,2,3,4,5,6,7,8,9}
    var a = ar[5:7] // 截断为{5,6}，len = 2、capacity = 5
    a = a[0:4] // 截断为{5,6,7,8}，len = 4、capacity = 5
```



# Map

## 特点

- 一种key-value的**无序**集合
- 数组和切片不能作为key
- 含有数组切片的结构体不能作为key，只包含内建类型的结构体可以作为key
- 指针和接口类型可以作为key
- 允许函数作为值，key用来选择要执行的函数（看下面代码，注释1处）
- 切片可以作为值（看下面代码，注释2处）
- 可以动态伸缩，不存在固定长度或最大限制，make的时候可以指定map的初始容量capacity
- 引用类型，不是值类型

```go
func main() {
    // 1. int作为key，func() int作为value
	mf := map[int]func() int{
		1: func() int { return 10 },
		2: func() int { return 20 },
		5: func() int { return 50 },
	}
	fmt.Println(mf)
}
// 2. int作为key，[]int切片作为value
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```



## 声明&初始化

```go
// 声明方式
var map1 map[int]string

// 初始化方式
var map1 = make(map[keytype]valuetype)
map2 = map[string]int{"one": 1, "two": 2}
```

> 不要使用new来构建map，否则会获得一个空引用得指针，相当于声明了一个未初始化得变量，并获取了它的地址



## for - range用法

```go
for key, value := range map1 {
	...
}
```



# 结构体

## 定义方式

```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

> 结构体得字段可以是任何类型，甚至是结构体本身，也可以是函数或者接口



## 初始化方式

```go
type Point struct { x, y int }
```



### 使用new函数

![image-20200716163446428](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716163446428.png)

> new函数获取到得是结构体类型得指针



### 获取结构体类型指针
![image-20200716163509504](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716163509504.png)

> 使用new函数跟使用&T{}是等价的，都是产生结构体类型指针



### 获取结构体类型
![image-20200716163527974](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716163527974.png)



### 混合字面量语法

```go
point1 := Point{0, 3}  (A)
point2 := Point{x:5, y:1}  (B)
point3 := Point{y:5}  (C)
```

> 结构体的类型定义在它的包中必须是唯一的，结构体的完全类型名是：packagename.structname



## 点号选择器

- 点号选择器可以给结构体的字段赋值

```go
    point.x = 5
```

- 点号选择器可以获取结构体字段的值

```go
    x := point.x
```

- 无论是结构体类型还是结构体类型指针，都可以使用点号来引用结构体字段

```go
    type myStruct struct { i int }
    var v myStruct    // v是结构体类型变量
    var p *myStruct   // p是指向一个结构体类型变量的指针
```



## 内存布局

![image-20200716163617228](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200716163617228.png)

> 结构体和它所包含的数据在内存中是以连续块的形式存在的



## 结构体工厂函数

> Go不支持面向对象编程语言中的构造方法，但是可以通过函数实现

```go
type File struct {
    fd      int     // 文件描述符
    name    string  // 文件名
}

// 定义工厂方法，函数名大写字母开头才能被跨包调用
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    return &File{fd, name}
}

// 调用工厂方法
f := NewFile(10, "./test.txt")
```

> 如果想要强制使用工厂函数，那么可以**将结构体的类型改为首字母小写**



## 结构体的标签

> 结构体中的字段，除了名字和类型，还有一个可选的标签，它是附属在字段的字符串（相当于字段的解释）

标签的内容需要通过反射获取

```go
type TagType struct { // tags
	field1 bool   "An important answer"
	field2 string "The name of the thing"
	field3 int    "How much there are"
}

func main() {
	tt := TagType{true, "Barak Obama", 1}
	for i := 0; i < 3; i++ {
		refTag(tt, i)
	}
}

func refTag(tt TagType, ix int) {
    // 使用reflect包
	ttType := reflect.TypeOf(tt)
	ixField := ttType.Field(ix)
	fmt.Printf("%v\n", ixField.Tag)
}
```



## 匿名字段和内嵌结构体

结构体可以包含一个或多个匿名字段，此时字段的类型就是字段的名称

```go
// 字段名称分别是：bool, string, int
type TagType struct {
	bool
	string
	int
}
```
> 在结构体中，对于每种数据类型，只能有一个匿名字段

匿名字段本身也可以是一个结构体类型

> Go语言的继承是通过内嵌或组合来实现的



## 结构体属性跨包调用

> 结构体的属性首字母大写，遵循大小写可见性规则

```go
type TagType struct {
	Field1 bool
	Field2 string
	Field3 int
}
```



# 方法

- Go的方法是**作用在接收者上**的一个函数，接收者是某种类型的变量，因此方法是一种特殊的函数
- **接收者不能是接口类型**，因为接口是一个抽象定义，但是方法是一个具体实现
- 接收者是指针类型变量的时候，可以改变接收者变量的值



## 方法定义

```go 
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

- 在方法名之前，func关键字之后的括号中指定接收者
- recv类似于面向对象语言中的this或self，Go中并没有这两个关键字



## Go中的面向对象

- 结构体 + 方法 = 面向对象中的类
- 结构体和方法可以放在不同的源文件，但是必须在同一个包

> 如果想要打破这条规则，可以**为类型定义别名，然后再为别名定义方法**，这样就可以不在同一个包下

- 类型T（或者\*T）上的所有方法的集合叫做类型T（或\*T）的方法集



## 方法不允许重载

> 方法实际上也是函数，函数不能重载

- 基于接收者类型的方法重载是允许的

```go
// 接收者类型不一样的时候，允许方法重载
func (receiver receiverType1) func1 (args... argType) returnType 
func (receiver receiverType2) func1 (args... argType) returnType 
```



## 非结构体类型变量调用方法的实现方式

```go

// 1. 给非结构体类型定义别名
type IntVector []int

// 2. 创建方法
func (v IntVector) sum() (s int){
    for _, x := range v {
        s += x
    }
    return
}

func main(){
    // 3. 调用方法
    IntVector{1,2,3}.sum();
}

```



## 函数和方法的区别

- 函数将变量作为参数
- 方法在变量上被调用
- 接收者类型跟方法必须在同一个包中



# 接口

> Go语言不是传统的面向对象编程语言，**没有类和继承的概念**，但是有非常灵活的接口，可以实现面向对象的特征，接口提供一种方式来说明对象的行为



## 定义

```go
    type Namer interface {
        Method1(param_list) return_type
        Method2(param_list) return_type
        ...
    }
```

## 命名方式

- 方法名 + er后缀（接口只有一个方法的时候）
- 接口以able结尾
- 以大写字母I开头



## 实现规则

- 接口可以被隐式实现，多个类型可以实现同一个接口
- 实现接口的某个类型还可以有其他的方法
- 一个类型可以实现多个接口

```go
    // 1. 定义一个接口
    type Shaper interface {
    	Area() float32
    }
    
    type Square struct {
    	side float32
    }
    
    // 2. Square类型指针实现接口中的方法
    func (sq *Square) Area() float32 {
    	return sq.side * sq.side
    }
    
    type Rectangle struct {
    	length, width float32
    }
    
    // 3. Rectangle类型实现接口中的方法
    func (r Rectangle) Area() float32 {
    	return r.length * r.width
    }
    
    func main() {
    
    	r := Rectangle{5, 3} 
    	q := &Square{5}
    	// 4. 通过接口数组接收两个变量
    	shapes := []Shaper{r, q}
    	fmt.Println("Looping through shapes for area ...")
    	for n, _ := range shapes {
    		fmt.Println("Shape details: ", shapes[n])
    		// 5. 实现多态
    		fmt.Println("Area of this shape is: ", shapes[n].Area())
    	}
    }
```



## 接口嵌套

一个接口可以包含多个接口，相当于将内嵌接口的方法列举在外层接口

```go
// 1. 定义接口ReadWrite和Lock
type ReadWrite interface{
    Read(b Buffer) bool
    Write(b Buffer) bool
}
type Lock interface{
    Lock()
    Unlock()
}
// 2. 在接口中添加了两个匿名字段：ReadWrite和Lock，实现接口嵌套
type File interface{
    ReadWrite
    Lock
    Close()
}
```



## 接口变量类型断言

> 接口类型变量可以包含任何类型的值，所以需要在运行时检测变量中存储的值的实际类型

```go
    t := varI.(T)
    // 或者可以使用下面这种方式
    if t, ok := varI.(T); ok {
        Process(t)
        return
    }
```

> varI必须是一个接口类型变量



## 类型判断（type - switch）

> 接口变量的类型也可以使用switch来检测

```go
switch t := areaIntf.(type) {
    case *Square:
    fmt.Printf("Type Square %T with value %v\n", t, t)
    case *Circle:
    fmt.Printf("Type Circle %T with value %v\n", t, t)
    case nil:
    fmt.Printf("nil value: nothing to check?\n")
    default:
    fmt.Printf("Unexpected type %T\n", t)
}
```

> type - switch中不允许有fallthrough



## 接口方法集调用规则

- 类\*T可调用方法集包括接收者为\*T和T的所有方法集
- 类型T的可调用方法集包含接收者T的所有方法，但不包含接收者为\*T的方法



## 空接口

```go
    type Any interface{}
```

- 空接口是不包含任何方法的接口
- 任何类型的值都可以赋值给空接口类型变量



# 面向对象

> Go没有类，而是松耦合的类型、方法对接口的实现



## 面向对象三大特征

- 封装
- 继承
- 多态



## Go中的面向对象特征

- 封装：Go对数据的访问控制简化为两层
    - 包范围内，通过标识**首字母小写**，对象只在它所在的包内可见
    - 可导出的，通过标识**首字母大写**，对象在包外也可见
- 继承：通过组合实现，**内嵌多个类型可以实现多重继承**
- 多态：通过接口实现，某个类型的实例可以赋值给它所实现的任意接口类型的变量



# 【todo】协程





# channel

> 负责协程之间的通信，从而避免所有由共享内存导致的陷阱
>
> 通道只能传输一种类型的数据（任意一种类型）



## 声明方式

```go
var ch1 chan string // 声明一个字符串通道
ch1 = make(chan string) // 实例化通道
```



## 通信操作符 

标识数据的传输，数据按照箭头的方向流动

```go
// 往通道发送数据
ch <- i 
```

for循环从通道中获取数据

```go
for v := range ch {
    fmt.Printf("The value is %v\n", v)
}
```

## 指定通道的方向

```go
    var send_only chan<- int 		// 只接收数据的通道
    var recv_only <-chan int		// 只发送数据的通道
```



## 关闭通道

- 通道是可以被显示关闭的；只有发送方需要关闭通道，接收方不需要关闭通道
- close函数关闭通道
    - 将通道标记为无法通过发送操作<\-接受更多的值；给已经关闭的通道发送或者再次关闭都会导致运行时的 panic 
- 检测通道是否关闭

```go
    v, ok := <-ch   // 使用, ok操作符检测通道是否关闭
```



## 协程切换

```go
    select {
    case u:= <- ch1:
           ...
    case v:= <- ch2:
            ...
            ...
    default: // no value ready to be received
            ...
    }
```

- 通过select关键字，从不同的并发执行的协程中获取值
- select关键字可以监听进入通道的数据或从通道出去的数据
- select 要做的事，选择处理列出多个通信情况中的一个
    - 如果都阻塞了，会等待知道其中一个可以处理
    - 如果多个可以处理，随机选择一个
    - 如果没有通道操作可以处理，但写了default语句，它就会执行default（确保不被阻塞）