# Go Interface
## Duck Typing

>If it walks like a duck and it quacks like a duck, then it must be a duck.

[维基百科的定义](https://en.wikipedia.org/wiki/Duck_typing?fileGuid=6Tt8PWVRXWYdcTvC)

## Why

面向对象编程的四个特性，封装、抽象（是否能成为特性之一存在争议）、继承、多态。

**Go** 语言中的封装和继承是通过 **struct** 实现，而多态则是 **interface** 通过 **DuckTyping** 思想来实现的。

接口是计算机系统中多个组件共享的边界，不同的组件能够在边界上交换信息。帮助我们解除上下游的耦合，隐藏底层实现。

## 特点

### 类型

**interface** 是一种类型。

### 隐式接口

不同于 **Java** 等其他语言中的接口需要显式地 **Implements**

**Go** 语言中如果一个类型实现了一个 **interface** 中所有方法，我们说该类型实现了该 **interface**。

### 空 interface

* **runtime.iface** 带一组方法的接口
* **runtime.eface** 不含任何方法的接口，但并不表示任意类型
## 示例

```go
package main

import (
    "fmt"
    "math"
)

type geometry interface {
    area() float64
    perim() float64
}

type rect struct {
    width, height float64
}

type circle struct {
    radius float64
}

func (r rect) area() float64 {
    return r.width * r.height
}

func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}

func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c circle) perim() float64 {
    return 2 * math.Pi * c.radius
}

func measure(g geometry) {
    fmt.Println(g)
    fmt.Println(g.area())
    fmt.Println(g.perim())
}

func main() {
    r := rect{width: 3, height: 4}
    c := circle{radius: 5}
    measure(r)
    measure(c)
}
```
## 指针与接口

**Go** 语言中同时使用指针和接口时会发生一些让人困惑的问题。

```go
type Cat struct {}

type Duck interface { ... }

func (c  Cat) Quack {}  // 使用结构体实现接口
func (c *Cat) Quack {}  // 使用结构体指针实现接口
var d Duck = Cat{}      // 使用结构体初始化变量
var d Duck = &Cat{}     // 使用结构体指针初始化变量
```

无法编译的例子

```go
type Duck interface {
	Quack()
}

type Cat struct{}

func (c *Cat) Quack() {
	fmt.Println("meow")
}

func main() {
	var c Duck = Cat{}
	c.Quack()
}

$ go build interface.go
./interface.go:20:6: cannot use Cat literal (type Cat) as type Duck in assignment:
	Cat does not implement Duck (Quack method has pointer receiver)
```
**Go** 语言在传递参数时都是 值传递。

对于 **Cat{}** 来说，这意味着 **Quack** 方法会接受一个全新的 **Cat{}**，因为方法的参数是 ***Cat**，编译器不会无中生有创建一个新的指针。即便编译器可以创建新指针，这个指针指向的也不是最初调用该方法的结构体。


---


## 相关资料

* [Go 语言设计与实现-接口 by draveness](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/?fileGuid=6Tt8PWVRXWYdcTvC)
* [计算机接口 wiki](https://en.wikipedia.org/wiki/Interface_(computing)?fileGuid=6Tt8PWVRXWYdcTvC)
* [如何优雅的使用 Go 接口](https://zhuanlan.zhihu.com/p/63219494?fileGuid=6Tt8PWVRXWYdcTvC)
* [Go by Example: Interface](https://gobyexample.com/interfaces?fileGuid=6Tt8PWVRXWYdcTvC)
* [How Interfaces Work in Go](https://www.tapirgames.com/blog/golang-interface-implementation?fileGuid=6Tt8PWVRXWYdcTvC)
