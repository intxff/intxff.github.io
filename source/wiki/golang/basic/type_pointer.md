---
layout: wiki
wiki: golang
title: 指针和非安全指针
order: 68
---

## 值尺寸

编译器决定，至少为 1 word，否则没办法寻址所有内存

## 字面量

&可取址值

## 零值

`nil`

## 内置安全指针的限制

* 不可算术运算
* 指针类型之间不可随意转换
* 指针值不可随意比较
    * \*T(A) == nil
    * \*T(A) 和 \*T(B)
    * \*T1(A) 和 \*T2(B)，\*T1 和 \*T2 必须能够隐式转换
* 指针值不可随意赋值给另一个指针值，必须可以比较才能赋值

## 非安全指针 unsafe.Pointer

```go
package unsafe
type Pointer *ArbitraryType
func Alignof(variable ArbitraryType) uintptr
func Offsetof(selector ArbitraryType) uintptr 
// uintptr 只是一个整数值，不是指针
//selector 不能有隐式的类型内嵌的指针匿名字段
// type S struct {y int}
//var x struct {
//    int
//    *S
//}
// x.y 必须为 x.S.y 才可以传入 selector
func Sizeof(variable ArbitraryType) uintptr
func Add(ptr Pointer, len IntegerType) Pointer
func Slice(ptr *ArbitraryType, len IntegerType) []ArbitraryType
```

unsafe.Pointer 可以与任意指针转换

### 正确的使用模式 

1. `\*A` 类型转 `\*B` 类型(转换要有意义，且 sizeof A >= sizeof B)
2. `\*T` 转换为 `uinptr` (不用，垃圾方法)
3. `\*T` 转换为 `uintptr` 进行计算再转回 `\*T`
> 注意 uintptr 仅仅只是值，垃圾回收不考虑这个值，所以要生成即用
> v := (\*int)(unsafe.Pointer(uintptr(unsafe.Pointer)+N+M-X-Y))
> 下面分开了垃圾回收 x 值所对应的内存就会出现 bug
> x := uintptr(unsafe.Pointer)+N+M-X-Y)
> v := (\*int)(unsafe.Pointer(x)
4. unsafe.Pointer 转换为 uinptr 给 Syscall (仅能给 syscall)
5. 将 reflect 包内函数或方法返回的 uinptr 立即转换为 unsafe.Pointer (理由跟 3 一样)
6. 将一个reflect.SliceHeader或者reflect.StringHeader值的Data字段转换为非类型安全指针，以及其逆转换。
