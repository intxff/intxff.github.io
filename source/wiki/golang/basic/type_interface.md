---
layout: wiki
wiki: golang
title: interface
order: 73
---
```go
// 空接口值
type _interface struct {
    dynamicType  *_type         // 引用着接口值的动态类型
    dynamicValue unsafe.Pointer // 引用着接口值的动态值
}
// 非空接口值
type _interface struct {
   dynamicTypeInfo *struct {
      dynamicType *_type       // 引用着接口值的动态类型
      methods     []*_function // 引用着动态类型的对应方法列表
   }
   dynamicValue unsafe.Pointer // 引用着动态值
}
```
接口类型定义了一个方法集，任何实现了这一方法集的类型，都实现了这一接口，其值可以作为接口值被赋值给此接口类型量

## 值尺寸
2 words

## 字面量
任何实现接口的类型的字面量都可以作为值赋予

## 零值
`nil`
> 注意下面情况的区分
> type A interface{}
> var a, b A
> var c []int
> b = c
> a, c 为 nil，但是 b 不为 nil，b 是动态值为 nil 的非零值空接口

## 接口的术语
* 实现: 某类型的方法集是接口的超集
* 动态值和动态类型: 接口值所绑定的类型和值

## 绑定动态值
* 动态值为非接口: 非接口值必须实现了接口，复制非接口值到动态值，并创建接口值的动态类型和方法列表
* 动态值为接口: 此接口值必须实现了接口，复制接口值的动态值，并创建此接口值的动态值的动态类型和方法列表
> 由此可见，对于大的数据结构，使用指针作为动态值可以更为高效

## 多态和反射
观察非空接口的类型定义，其内部有动态值、动态类型及其方法集。
* 反射: 依靠动态类型及动态值实现
* 多态: 动态类型的方法集(多种类型都可以实现接口，同一个方法名称对应类型不同，其调用形式不变)

## 接口类型内嵌
内嵌的接口等于直接展开，不能内嵌自己(废话)

## 同类型接口的值比较

* 接口值和非接口值比较: 非接口值隐式转换为接口值
* 接口值和接口值比较:
    * 动态类型为不可比较类型， panic
    * 动态类型相同，按照动态类型的规则比较，值相同则两接口值相同

## 其它

### []T 与 []I，T 实现了接口类型 I
### 一个接口类型每个指定的每一个方法都对应着一个隐式声明的函数
### 空接口作为参数
虽然空接口类型可以接收任何值作为绑定值，但是空接口值的比较应该按上面的方式比较，由此也会有反直觉的两个 nil 值不等的问题
```go
package main

import (
	"fmt"
)

type A interface {
}

func do(i interface{}, t interface{}) {
	if i != nil {
		fmt.Println("i not nil")
	}
	if i != t {
		fmt.Println("i != t")
	}
}

func main() {
	var a []int
	var b map[int]string
	do(a, b)
}
```

```golang
package main
import "fmt"
type MyImplement struct{}
func (m *MyImplement) String() string {
    return "hi"
}
func GetStringer() fmt.Stringer {
    var s *MyImplement = nil
    return s 
    // s 赋值给 fmt.Stringer 接口类型值作为返回值，返回值变为 fmt.Stringer 的接口值
    // 其绑定的动态值为 nil，动态类型为 *MyImplement，但是不是 fmt.Stringer(nil) 值
}
func main() {
    if GetStringer() == nil {
        fmt.Println("GetStringer() == nil")
    } else {
        fmt.Println("GetStringer() != nil") // 输出这个
    }
}
```
