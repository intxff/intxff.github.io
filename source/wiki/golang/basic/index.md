---
layout: wiki
wiki: golang
title: 语法速览
order: 1
---

## 词法(lexical)分类

### 关键字(keyword)

```go
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### 标识符(identifier)
* `_` 作为标识符：`_ := 1` 丢弃值，`import _ "fmt"` 防止尚未使用的包导致编译出错
* 可导出量首字母大写

### 运算符和标点符号(operator and punctuation)

```go
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

### 字面量(literal)

## 包和加载顺序

### 声明包
```go
package 包名
```

### 导入

```go
import [包名] 包
import (
    [包名] 包
    [包名] 包
    [包名] 包
)
```

### `init()` 函数和包加载顺序
* 同一包中可以有多个 `init()` 函数，但是后者会覆盖前者
* main 包会先递归的加载依赖包中的导出量和 `init()` 函数，最后才是本身

## 注释

单行注释: `//`
多行注释: `/* */`

## 常量(constant)和变量(variable)

### 变量声明和定义
局部声明的变量必须要被使用至少一次
* var 关键字声明和定义
    这种方式可以在任何作用域使用
    ```go
    var a, b Type // 同一类型可以并列，仅声明
    var a, b Type = xxx, xxx // 同一类型可以并列，声明并定义
    var a string, b int = "123", 1 //没有这种
    // 列表型
    var (
        a, b Type1
        c    Type2
    )
    var (a, b Type1;c Type2)
    var a = 1 // 这种是合法的，类型推断赋值时赋予 a 字面量 1 的默认类型
    // 下面这种不存在，不同于 struct 的省略
    var (
        a, b int
        c
    )
    struct {
        a, b int
        c //c -> int type
    }
    ```
* 短变量声明: `a := xxx`，`a, b := xxx, yyy`
    仅可以用在局部作用域中，属于简单语句

### 常量声明和定义

常量必须是 bool, number, string 值，常量会在编译时替换为值

* 字面量常量(无名常量)
* 有名常量

    * 类型确定: 其值不能溢出类型限定的范围
    * 类型不确定: 其值可以溢出其默认类型的范围

    ```go
    const (
        z string = "123" // 类型确定有名常量
        a = 123 // 类型不确定有名常量，类型推断为 int
        b
        c // b 和 c 都是 int(123)
    )
    ```

## 表达式、语句和简单语句

### 表达式
表达式是通过函数调用和运算符对值进行计算的描述
* 常量表达式: 常量运算，编译期确定值
* 有返回值的函数调用和方法调用
* 显式类型转换: `int32(114514)`
* 类型断言: `x.(int)`
* 选择器: `p.m()`
* 索引和取切片操作: `array[0]` `slice[:]`

### 语句
* 终止语句
* 空语句(简单语句)
* 标签语句
* 表达式语句(带返回值的函数活方法调用、通道接收操作属于简单语句)
* 发送操作语句(简单语句)
* ++/-- 语句(简单语句)
* 赋值语句
    * 变量短声明(简单语句)
    * 纯赋值语句(简单语句)
* if
* switch
* for
* go
* select
* return
* break/continue/fallthrough
* defer
* goto

## 流程控制
所有语句的 `{` 不能另起一行

### 循环语句
* 传统 for 循环

```go
for 简单初始化语句;条件;简单更新语句 { 
}
for 条件 {
}
for {
}
```

作用域要注意，比如

```go
for i := 0; i < 10; i++ {
    // 此时 i 相当于外层，后续使用 i， 就类比于函数内的局部作用域使用
    // 全局作用域内的数据
    i := i  // 所以这样使用是合法的，且这个 i 与外层的 i 不同了
    // 主要用于下面的闭包情况，如果 i 不作为传参复制到协程的栈中，
    // i 必须要先行复制到协程引用的空间中
    go func() {
        dosomething(i)
    }()
    // 下面的结果跟上面一样，传参方式保证协程中 i 的值
    go func(i type) {
        dosomething(i)
    }(i)
} */
```
其它流程控制带有的局部作用域也满足这一情况

* for range 迭代容器类型
```go
for key, value := range Container {
}
// 相当于
temp := Container
var key Type
var value Type
for i := 0; i < len(temp); i++ {
    key = index of temp
    value = temp[key]
    dosomething(key, value)
}
```
所以有以下事实
* key 和 value 在每次循环时都是同一个变量，只是值被更新了
* 被循环的量只是原量的一个浅复制
* key 和 value 的值也是一个浅复制

### 条件语句

#### if else
```go
if 简单语句初始化; 条件 {
} else if xxxx {
} else {
}
```

#### switch case
```go
switch 简单语句初始化; 比较对象0 {
case 比较对象列表1:
    dosomething
case 比较对象列表2:
    dosomething
    fallthrough // dosomething 结束后执行下一 case 的 dosomething 
    // fallthrough 必须是当前 case 的最后一条语句，放在最后一条 if 语句中也不行
    // fallthrough 不能在最后一个 case 中
    .
    .
    .
case 比较对象列表N:
    dosomething
default:
}
```

#### type switch
``` go
switch 简单语句; v := x.(type) {
case typelist0:
    dosomething
case typelist1:
    dosomething
    .
    .
default: 
}
// 下面这个等于多个 if 判断
switch {
case 判断式0:
    dosomething
case 判断式1:
    dosomething
    .
    .
default:
    dosomething
}
```

#### select case
``` go
select {
case ChannelOperation0:
    dosomething
case ChannelOperation1:
    dosomething
    .
    .
default:
    dosomething
}
```

### goto 跳转

## 函数和方法

### 函数

#### 有名函数
```go
func 函数名(入参列表) (出参列表) {
函数体
}
```
函数名应该当作“常量”的一样的事物，其类型是函数签名(function signature) `func (in parameters) (out parameters)`
#### 匿名函数
```go
x := func(入参列表) (出参列表) {}
```
#### 入参列表和出参列表
* 函数传参都是传值，即使是指针作为参数，函数内使用的依然是指针的复制。
* 入参必须具名，出参不一定要具名，出参声明的变量不能在函数体中再次声明，除非出参列表不具名。
``` go
f := func(x int,y int) (int) {
    z := x+y
    return z // 必须指明 return z
    // 如果 z 是返回 &z，在 go 中这种是合法的，go 会自动将局部变量 z 逃逸到堆上(现象就是函数内的局部变量地址跟传出的结果的地址是一样的)
}
f := func(x,y int) (z int) {
    z := x+y //错误
    z = x+y
    return //啥也没有就相当于 return z
}
```
* 入参数目可变
```go
func a(x string, y ...int) (z int) {
    fmt.Println(x)
    for _, i := range y {
        z += i
    }
    return
}
```

### 方法

#### 可以声明方法的类型的要求
如果有类型 T 以及 \*T 要声明方法
* 方法和类型必须位于同一个包中(这也是为什么不能给标准库内置基本类型声明方法)
* T 必须是定义类型
* T 不能是指针和接口类型

#### 形式
```go
type T anytype

func (属主) 方法名(入参列表) (出参列表) {
}
```

#### go 的语法糖
对方法的调用等于对其隐式函数的调用。

```go
type T []int
func (a T)dosomething(x M, y N) {
    dosomething
}
//编译器同时自动声明下面的函数和方法
//func T.dosomething(a T, x M, y N){}
//func (a *T)dosomething(x M, y N)
//func *T.dosomething(a *T, x M, y N){}
//所以下面的调用
a := T{xxxxx}
b := &T{xxxxx}
a.dosomething(x,y)
//后续 3 个可以直接使用，即使并没有显式声明
T.dosomething(a,x,y)
b.dosomething(x,y)
*T.dosomething(&b,x,y)

func (a *T)dosomething1(x M, y N) {
    dosomething
}
// 这个仅会声明一个
// func *T.dosomething1(a *T, x M, y N){}
```

为什么会这样，如果有个指针值 v 是 \*T 类型，那么我解引用 \*v 获得的值一定可以调用类型 T 的方法，也就是说，编译器要保证每个为 T 声明的方法，都要再声明个 \*T 的方法以保证上述成立。这样 T 的方法就多了一个方法，同时对方法的调用就是对隐式函数的调用，所以又会生成两个函数。

### 内置函数

* 部分内置函数调用是在编译器确定值 `len(T)` `cap(T)` ，T 是数组或者数组指针
* 大部分内置函数 (除了 `recover()` 和 `copy()`) 的值不能被丢弃
* 有返回值的函数调用是个表达式

### defer 函数
* 在函数的退出阶段执行收尾工作，可以改变返回值
    * 正常执行完 `return` 退出
    * `panic()` 退出
    * `runtime.GOEXIT()` 退出
* 以栈的顺序执行，先入栈的后执行
* defer 和 recover 的组合
    ```go
    func dosomething() {
        defer func() {
            recover()
        }
        panic()
    }
    ```
下面三种情况都不会生效，仅有上述情况 `recover` 能恢复 `panic`。
    * 没有 `defer func` 直接 `defer recover()` 
    * `recover()` 与 `panic()` 不是隔了一个 `defer func`
    * `defer func` 放在了 `panic` 之后(废话，panic 之后的语句都不会执行)

## go 协程
``` go
//go 函数调用或匿名函数调用
go dosomething()
go func() {
    dosomething
}()
```
