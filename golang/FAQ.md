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

**举例：**

1.**写已经关闭的 chan**

```go
func main(){
    c := make(chan int,3)
    close(c)
    c <- 1
}
//输出结果
panic: send on closed channel

goroutine 1 [running]
main.main()
...
```

- 注意这个 `send on closed channel`，待会会提到。

2.**读已经关闭的 chan**

```go
package main
import "fmt"

func main()  {
    fmt.Println("以下是数值的chan")
    ci:=make(chan int,3)
    ci<-1
    close(ci)
    num,ok := <- ci
    fmt.Printf("读chan的协程结束，num=%v， ok=%v\n",num,ok)
    num1,ok1 := <-ci
    fmt.Printf("再读chan的协程结束，num=%v， ok=%v\n",num1,ok1)
    num2,ok2 := <-ci
    fmt.Printf("再再读chan的协程结束，num=%v， ok=%v\n",num2,ok2)
    
    fmt.Println("以下是字符串chan")
    cs := make(chan string,3)
    cs <- "aaa"
    close(cs)
    str,ok := <- cs
    fmt.Printf("读chan的协程结束，str=%v， ok=%v\n",str,ok)
    str1,ok1 := <-cs
    fmt.Printf("再读chan的协程结束，str=%v， ok=%v\n",str1,ok1)
    str2,ok2 := <-cs
    fmt.Printf("再再读chan的协程结束，str=%v， ok=%v\n",str2,ok2)

    fmt.Println("以下是结构体chan")
    type MyStruct struct{
        Name string
    }
    cstruct := make(chan MyStruct,3)
    cstruct <- MyStruct{Name: "haha"}
    close(cstruct)
    stru,ok := <- cstruct
    fmt.Printf("读chan的协程结束，stru=%v， ok=%v\n",stru,ok)
    stru1,ok1 := <-cs
    fmt.Printf("再读chan的协程结束，stru=%v， ok=%v\n",stru1,ok1)
    stru2,ok2 := <-cs
    fmt.Printf("再再读chan的协程结束，stru=%v， ok=%v\n",stru2,ok2)
}

//输出结果
以下是数值的chan
读chan的协程结束，num=1， ok=true
再读chan的协程结束，num=0， ok=false
再再读chan的协程结束，num=0， ok=false
以下是字符串chan
读chan的协程结束，str=aaa， ok=true
再读chan的协程结束，str=， ok=false
再再读chan的协程结束，str=， ok=false
以下是结构体chan
读chan的协程结束，stru={haha}， ok=true
再读chan的协程结束，stru=， ok=false
再再读chan的协程结束，stru=， ok=false
```

**分析：**

1.**为什么写已经关闭的 chan 就会 panic 呢？**

在 `src/runtime/chan.go`中，当 `c.closed != 0` 则为通道关闭，此时执行写，源码提示直接 `panic`，输出的内容就是上面提到的 `"send on closed channel"`。

```go
//在 src/runtime/chan.go
func chansend(c *hchan,ep unsafe.Pointer,block bool,callerpc uintptr) bool {
    //省略其他
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }   
    //省略其他
}
```

2.**为什么读已关闭的 chan 会一直能读到值？**

```go
func chanrecv(c *hchan,ep unsafe.Pointer,block bool) (selected,received bool) {
    //省略部分逻辑
    lock(&c.lock)
    //当chan被关闭了，而且缓存为空时
    //ep 是指 val,ok := <-c 里的val地址
    if c.closed != 0 && c.qcount == 0 {
        if receenabled {
            raceacquire(c.raceaddr())
        }
        unlock(&c.lock)
        //如果接受之的地址不空，那接收值将获得一个该值类型的零值
        //typedmemclr 会根据类型清理响应的内存
        //这就解释了上面代码为什么关闭的chan 会返回对应类型的零值
        if ep != null {
            typedmemclr(c.elemtype,ep)
        }   
        //返回两个参数 selected,received
        // 第二个采纳数就是 val,ok := <- c 里的 ok
        //也就解释了为什么读关闭的chan会一直返回false
        return true,false
    }   
}
```

`c.closed != 0 && c.qcount == 0` 指通道已经关闭，且缓存为空的情况下（已经读完了之前写到通道里的值）

如果接收值的地址`ep`不为空：

- 那接收值将获得是一个该类型的零值
- `typedmemclr` 会根据类型清理相应地址的内存
- 这就解释了上面代码为什么关闭的 chan 会返回对应类型的零值

### 3.8Q：对未初始化的chan进行读写，会怎么样？为什么？

A：读写未初始化的chan都会阻塞

**举例：**

1.写未初始化的chan

```go
package main

func main() {
	var c chan int
	c <- 1
}

//输出结果
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send (nil chan)]:
main.main()
        D:/Golang-workspace/src/learning/Test/读写未初始化的chan/main.go:5 +0x3d
```

- 注意这个`chan send (nil chan)`，后面会提到

2.读未初始化的chan

```go
package main

import "fmt"

func main() {
	var c chan int
	num, ok := <-c
	fmt.Printf("读chan的协程结束，num=%v,ok=%v \n", num, ok)
}

//输出结果
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive (nil chan)]:
main.main()
        D:/Golang-workspace/src/learning/Test/读写未初始化的chan/main.go:7 +0x4d
```

注意这个`chan receive (nil chan)`，后面也会提到

**分析：**为什么对未初始化的chan读写就会阻塞呢？

1.对于写的情况

```go
//在 src/runtime/chan.go中
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	if c == nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}
    ...
}
```

- 未初始化的chan此时是等于nil，当它不能阻塞的情况下，直接返回false，表示写chan失败
- 当chan能阻塞的情况下，则直接阻塞`gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)`，然后调用`throw(s string)`抛出错误，其中`waitReasonChanSendNilChan`就是刚刚提到的报错信息`chan send (nil chan)`

2.对于读的情况

```go
//在 src/runtime/chan.go中
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	// raceenabled: don't need to check ep, as it is always on the stack
	// or is new memory allocated by reflect.
	...
	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}
    ...
}
```

- 未初始化的chan此时是等于nil，当它不能阻塞的情况下，直接返回false，表示读chan失败
- 当chan能阻塞的情况下，则直接阻塞`gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)`，然后调用`throw(s string)`抛出错误，其中`waitReasonChanReceiveNilChan`就是刚刚提到的报错信息`chan receive (nil chan)`

### 3.9Q：for循环select时，如果通道关闭会怎么样？如果select中的case只有一个，又会怎么样？

- for循环`select`时，如果其中一个`case`通道已经关闭，则每次都会执行到这个case
- 如果select里边只有一个case，而这个case被关闭了，则会出现死循环

总结：

- select中如果任意某个通道有值可读时，它就会被执行，其他被忽略
- 如果没有`default`语句，select将有可能阻塞，直到某个通道有值可以运行，所以select最好有一个default，否则将有一直阻塞的风险

