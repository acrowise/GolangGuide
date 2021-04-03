![go_monkey](golang面试题总结.assets/go_monkey.jpg)

[TOC]

# Golang面试题总结--制作人张茂凯

## 1.基础知识归纳

1. 使用表达式`new(Type)`将创建一个Type类型的匿名变量，初始化为`Type类型的零值`，然后返回`变量的地址，返回的指针类型为*Type`

   **面试题：new与make的区别**

   > new 的作用是初始化一个指向类型的指针(*T)
   >
   > new函数是内建函数，函数定义：`func new(Type) *Type`
   >
   > 使用new函数来分配空间。传递给`new` 函数的是一个**类型**，不是一个值。返回值是指向这个新分配的零值的**指针**。
   >
   > make 的作用是为 slice，map 或 chan 初始化并返回引用(T)。
   >
   > make函数是内建函数，函数定义：`func make(Type, size IntegerType) Type`
   >
   > · 第一个参数是一个类型，第二个参数是长度
   >
   > · 返回值是一个类型
   >
   > `make(T, args)`函数的目的与`new(T)`不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是*T）的一个初始化的（不是零值）的实例

2. `byte`和`uint8`没有区别，`rune`和`uint32`没有区别。因为uint8和uint32直观上让人以为这是一个数值，但是实际上，它也可以表示一个`字符`，所以为了消除这种直观上的错觉，就诞生了byte和rune这两个类型的别名。且对于`rune`关键字来说，其表示范围(-2<sup>31</sup>~2<sup>31-1</sup>)，比byte(-128~127)可以表示更多的字符，特别是中文字符。

   ##### 面试题：翻转含有中文，数字，英文字母的字符串

   ```go
   package main
   import "fmt"
   
   func reverse(s []rune) string {
   	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
   		s[i], s[j] = s[j], s[i]
   	}
   	return string(s)
   }
   
   func main() {
   	var src = "aa张111"
   	dst := reverse([]rune(src))
   	fmt.Println(dst)
   }
   ```

3. 对于map来说，读取元素，直接使用`map[key]`即可，如果key不存在，系统不会报错，会返回其`value-type的零值`

4. 使用`delete`函数删除map元素，如果key不存在，`delete函数会静默处理`，不会报错

5. 使用`defer只是延时调用函数`，此时传递给函数里的变量，不应该受到后续程序的影响，相当于在defer处对变量做了`快照`，多个defer的调用顺序类似`栈`，且`defer在return之后调用`

   ##### 面试题：defer和return的调用顺序

   ```go
   package main
   
   import "fmt"
   
   var name = "go"
   
   func getName() string {
   	defer func() {
   		name = "js"
   	}()
   
   	fmt.Printf("getName函数中的name值为%v \n", name) // go
   
   	return name
   }
   
   func main() {
   	myname := getName()
   	fmt.Printf("main函数中的name为%v \n", name)     // js
   	fmt.Printf("main函数中的myname为%v \n", myname) // go
   }
   ```

6. `recover`的使用必须在defer函数中才能生效

7. `panic`会导致整个程序退出，但是在退出前，若有defer延迟函数，还得执行完defer。且`defer在多个协程之间没有效果`，在子协程里触发panic，只能触发自己协程内的defer，而不能调用main协程里的defer函数

   ```go
   package main
   
   import (
   	"fmt"
   	"time"
   )
   
   func main() {
   	//这个defer不会执行
   	defer fmt.Println("in main")
   
   	//子协程
   	go func() {
   		//这个defer还得执行完
   		defer fmt.Println("in goroutine")
   		panic("出现错误")
   	}()
   	
       //给子协程留出能执行的时间
   	time.Sleep(2 * time.Second)
   }
   
   ```

8. 在golang中，`接口就是方法签名的集合`，当一个类型定义了接口中的`所有方法`，我们称它实现了该接口，接口指定了一个类型应该具有的方法，并由该类型决定如何实现这些方法。一个接口（老师）下，在不同对象（人）上的不同表现，就是多态。

9. 根据`变量声明的位置不同`，作用域可以分为以下四个类型：

   - `内置作用域`：不需要自己声明，所有的关键字和内置类型、函数都拥有全局作用域
   - `包级作用域`：必须函数外声明，在该包所有文件内都可以访问
   - `文件级作用域`：不需要声明，导入即可。一个文件中通过import导入的包名，只能在该文件内使用
   - `局部作用域`：在自己的语句块内声明，包括函数、for、if等语句块，或自定义的{}语句块形成的作用域

   以上四种作用域，从上往下，范围从大到小，为了表述方便，这里将范围大的作用域称为高层作用域，范围小的作用域称为低层作用域。总结几点：

   - 低层作用域可以访问高级作用域
   - 同一层级的作用域是相互隔离的
   - 低层作用域里声明的变量会覆盖高层作用域里面声明的变量

10. `缓冲信道`允许信道里存储一个或多个数据，这意味着，设置了缓冲区后，发送端和接收端可以处于`异步`状态

11. `无缓冲信道`在信道里无法存储数据，这意味着，`接收端必须先于发送端准备好`，以确保你发送完数据后立马有人接收，否则会造成发送端堵塞，原因就是信道中无法存储数据，即发送端和接收端是`同步`进行的

12. 遍历信道，可以使用for搭配range关键字，在range时，要确保信道是否处于关闭状态，否则循环会阻塞

13. `不要通过共享内存来通信，要通过通信来共享内存`

14. `sync.WaitGroup类型`中的方法：

    - `Add`：初始值为0，传入的值会往计数器上加，这里一般传入子协程的数量
    - `Done`：当某个子协程完成后，可调用此方法，会从计数器上减一，通常可以使用defer调用
    - `Wait`：阻塞当前协程，直到实例里的计数器归零，一般放在main方法中阻塞，直到所有的子协程全部调用完毕

    ```go
    package main
    
    import (
    	"fmt"
    	"sync"
    )
    
    var wg sync.WaitGroup
    
    func worker(x int) {
    	//计数器减一
    	defer wg.Done()
    
    	for i := 0; i < 5; i++ {
    		fmt.Printf("worker %d 在做第%d份工作 \n", x, i)
    	}
    }
    
    func main() {
    	//添加子协程数量 为2
    	wg.Add(2)
    
    	go worker(1)
    	go worker(2)
    
    	//阻塞main协程  直到所有的子协程全部执行完毕
    	wg.Wait()
    }
    ```

15. 面对并发问题，我们始终应该优先考虑使用信道，如果通过信道解决不了，不得不使用共享内存来实现并发编程的，那么就需要使用golang中的`锁机制`了。golang中的锁分为两种：一个叫`Mutex`，利用它可以实现`互斥锁`，另一个叫`RWMutex`，利用它可以实现`读写锁`。其中RWMutex将程序对资源的访问分为`读操作`和`写操作`，为了保证数据的安全，它规定了当有人还在读取数据（即读锁占用）时，不允许有人更新这个数据（即写锁会阻塞）为了保证程序的效率，多个人（线程）读取数据（拥有读锁）时，互不影响，不会造成阻塞，它不会像Mutex那样只允许有一个人（线程）读取同一个数据

## 2.常见类型源码分析

### 2.1深度解密Go语言之slice

[深度解密Go语言之slice](https://mp.weixin.qq.com/s/MTZ0C9zYsNrb8wyIm2D8BA)

### 2.2深度解密Go语言之map

[深度解密Go语言之map](https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA)

## 3.常见面试题

### 3.1Q：字符串转换成byte数组，会发生内存拷贝吗？有没有什么办法可以在转换时不发生拷贝呢？

A：字符串转换成切片，会产生拷贝，严格来说，`只要是发生类型强转都会发生内存拷贝`。转换后 [ ]byte 底层数组与原 string 内部指针并不相同，可以确定数据被复制。

参考：[golang无复制高效实现string与[]byte转换](https://jingyan.baidu.com/article/4f7d571213e9da1a2019271e.html)

```go
package main

import (
	"reflect"
	"unsafe"
)

func main() {
	/*不用发生拷贝*/
	s := "hello world"
	ssh := *(*reflect.StringHeader)(unsafe.Pointer(&s))
	b := (*[]byte)(unsafe.Pointer(&ssh))
	_ = b
}
```

> 解释：
>
> - `StringHeader`是string类型的底层结构
>
>   // StringHeader is the runtime representation of a string.
>
>   ```go
>   type StringHeader struct {
>   	Data uintptr //数据指针
>   	Len  int //长度
>   }
>   ```
>
> - `SliceHeader`是slice类型的底层结构
>
>   // SliceHeader is the runtime representation of a slice.
>
>   ```go
>   type SliceHeader struct {
>   	Data uintptr //数据指针
>    	Len  int //长度
>   	Cap  int //容量
>   }
>   ```
>
> 如果想要在底层转换二者，只需要把StringHeader的地址强转成SliceHeader就行，使用`unsafe`包。

### 3.2Q：能说说uintptr和unsafe.Pointer的区别吗？

- `unsafe.Pointer`只是单纯的通用指针类型，用于转换不同类型的指针，它`不可以参与指针运算`。
- `uintptr`是用于指针运算的，GC不会把uintptr当指针，也就是说uintptr无法持有对象，uintptr类型的目标会被回收
- unsafe.Pointer可以和 普通指针 进行相互转换
- unsafe.Pointer可以和 uintptr 进行相互转换

### 3.3Q：拷贝大切片一定比小切片代价大吗？

A：并不是，所有的切片大小都相同，由3.2可知，slice切片的底层结构为SliceHeader

```go
type SliceHeader struct {
	Data uintptr //数据指针
 	Len  int //长度
	Cap  int //容量
}
```

包含三个字段（一个uintptr，两个int）,切片中的第一个字是`指向切片底层数组的指针`，这是切片的存储空间；第二个字段是切片的`长度`；第三个字段是切片的`容量`。将一个slice变量分配给另外一个变量只会复制这三个字段的`机器字`长度。所以`拷贝大切片和小切片的代价是一样的`。大切片和小切片的区别无非是Len和Cap的大小。

### 3.4Q：知道Golang的内存逃逸吗？什么情况下回发生内存逃逸？

A：嗯，了解一些。在编译程序优化理论中，逃逸分析是一种`确定指针动态范围`的方法——分析在程序的哪些地方可以访问到指针。逃逸分析由编译器决定内存分配的位置（分配在`栈`中还是`堆`中），不需要程序员指定。Golang在`编译阶段确定逃逸`。

`栈`（操作系统概念，区别数据结构概念）：可以简单理解为一次函数调用内部申请到的内存，它们会随着函数的返回把内存还给系统，整个分配内存的过程通过栈的分配和回收都会迅速。

`堆`（操作系统概念，区别数据结构概念）：堆在内存分配中类似于往一个房间里摆放各种家具，家具的尺寸有大有小，分配内存时，需要找一块足够装下家具的空间再摆放家具。堆适合不可预知大小的内存分配。但是为此付出的代价是分配速度较慢，而且会形成内存碎片。需要GC来进行回收。

**逃逸分析的用处（为了性能）:**

- `最大的好处应该是减少gc的压力`，不逃逸的对象分配在栈上，函数返回时就回收了资源，不需要gc标记清除。
- 因为逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好
- 同步消除，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。

**能引起变量逃逸的到堆上的典型情况：**

- `在方法内把局部变量指针返回`局部变量原本应该在栈中分配，在栈中回收，但是由于返回时被外部引用，因此其生命周期大于栈，则溢出。

  ```go
  package main
  
  import "fmt"
  
  //最常见的情况 在方法内把局部变量指针返回
  func test() *int {
  	var a = 10
  	return &a //发生变量逃逸  从栈逃逸到堆
  }
  
  func main() {
  	p := test()
  	fmt.Println(*p) //fmt.Println(a ...interface{})打印的变量都会发生逃逸
  }
  ```

  通过`go build -gcflags "-m -l"`进行逃逸分析：

  ```go
  .\main.go:7:6: moved to heap: a
  .\main.go:13:13: ... argument does not escape
  .\main.go:13:14: *p escapes to heap
  ```

- `栈空间不足逃逸（空间开辟过大）`

  ```go
  package main
  
  func main() {
  	//实际上当栈空间不足以存放当前对象时或无法判断当前切片长度时会将对象分配到堆中。
  	s := make([]int, 1000, 1000)
  	// s := make([]int, 100000, 100000)
  
  	for index, _ := range s {
  		s[index] = index
  	}
  }
  ```

  在切片大小为1000时，进行逃逸分析：

  ```go
  .\main.go:17:11: make([]int, 1000, 1000) does not escape
  ```

  增大切片的长度到100000，再进行逃逸分析:

  ```go
  .\main.go:18:11: make([]int, 100000, 100000) escapes to heap
  ```

- `动态类型逃逸`很多函数参数为interface类型，比如fmt.Println(a …interface{})，编译期间很难确定其参数的具体类型，也能产生逃逸。

- `对象指针被多个子程序（如线程 协程）共享使用`

### 3.5Q：怎么避免逃逸分析？

A：在`runtime/stubs.go:133`有个函数叫`noescape`。noescape可以在逃逸分析中隐藏一个指针，让这个指针在逃逸分析中`不会被检测为逃逸`。

```go
// noescape hides a pointer from escape analysis.  noescape is
// the identity function but escape analysis doesn't think the
// output depends on the input.  noescape is inlined and currently
// compiles down to zero instructions.
// USE CAREFULLY!
//go:nosplit
func noescape(p unsafe.Pointer) unsafe.Pointer {
	x := uintptr(p)
	return unsafe.Pointer(x ^ 0)
}
```

noescape()函数的作用是遮蔽输入和输出的依赖关系，是编译器不认为p会通过x逃逸，因此uintptr()产生的引用时编译器无法理解的。

### 3.6Q：reflect（反射包）如何获取字段tag？为什么json包不能导出私有变量的tag？

即Question：json包使用的时候，会在结体字段边加上加tag，有没有什么办法可以获取到这个`tag`的内容呢？

A：`tag`信息可以通过`反射（reflect包）`内的方法获取，下面通过一个栗子加深印象：

```go
package main

import (
	"fmt"
	"reflect"
)

type T struct {
	a string //小写无tag
	b string `json:"B"` //小写+tag
	C string //大写无tag
	D string `json:"D" other:"good"` //大写+tag
}

func printTag(st interface{}) {
	elem := reflect.TypeOf(st).Elem() //获取指针指向的值对应的结构体内容
	for i := 0; i < elem.NumField(); i++ {
		fmt.Printf("结构体内第%v个字段 %v 对应的json tag = %v,对应的other tag = %v\n", i+1, elem.Field(i).Name, elem.Field(i).Tag.Get("json"), elem.Field(i).Tag.Get("other"))
	}
}

func main() {
	t := T{
		a: "1",
		b: "2",
		C: "3",
		D: "4",
	}
	printTag(&t)
}
```

`json包`不能导出`私有变量的tag`是因为`取不到反射信息`，但是直接取`t.Field(i).Tag.Get("json")`却可以获取到私有变量的json字段，是为什么呢？

其实准确的说法是，json包里不能导出私有变量的tag是因为json包里认为私有变量为不可导出的`Unexported`，所以跳过获取名为json的tag的内容。具体可以看`/src/encoding/json/encode.go:1209`的代码。

```go
// typeFields returns a list of fields that JSON should recognize for the given type.
// The algorithm is breadth-first search over the set of structs to include - the top struct
// and then any reachable anonymous structs.
func typeFields(t reflect.Type) []field {
    // 注释掉其他逻辑...
    // 遍历结构体内的每个字段
    for i := 0; i < f.typ.NumField(); i++ {
        sf := f.typ.Field(i)
        isUnexported := sf.PkgPath != ""
        // 注释掉其他逻辑...
        if isUnexported {
            // 如果是不可导出的变量则跳过
            continue
        }
        // 如果是可导出的变量（public），则获取其json字段
        tag := sf.Tag.Get("json")
        // 注释掉其他逻辑...
    } 
    // 注释掉其他逻辑... 
}
```

### 3.7Q：对已经关闭的chan进行读写会怎么样？为什么？

A：读`已经关闭`的chan能一直读到东西，但是读到的内容根据通道内`关闭前是否有元素`而不同

- 如果chan关闭前，buffer（即带有缓冲的chan）内`还有元素未读`，会正确读到chan内的值，且返回的第二个bool值未true
- 如果chan关闭前，buffer内的元素已经被`读完`，chan内无值，接下来所有接收的值都会非阻塞直接成功，返回channel元素的`零值`，但是第二个bool值一直为false
- 写已经关闭的chan会panic



## 4.常见算法题

## 5.其他

### 5.1Redis相关

### 5.2MySQL相关

### 5.3操作系统相关

### 5.4计算机网络相关

### 5.5Git相关

### 5.6Docker相关

### 5.7Kubernetes相关

### 5.8常见共识算法

#### 5.8.1raft协议

#### 5.8.2PBFT（Practical Byzantine Fault Tolerance，实用拜占庭容错）

## 参考

公众号:

- Go语言中文网
- golang小白成长记
- 码农桃花源

网址：

-  [https://www.jianshu.com/p/8e4bbe7e276c](https://www.jianshu.com/p/8e4bbe7e276c)

- Ongaro D, Ousterhout J. In search of an understandable consensus algorithm［C］// USENIX Annual Technical Conference. [s.l.]: USENIX. 2014: 305-319．

- [https://www.cnblogs.com/aibabel/p/10973585.html](https://www.cnblogs.com/aibabel/p/10973585.html)

- Raft原理动画：[http://thesecretlivesofdata.com/raft/](http://thesecretlivesofdata.com/raft/)