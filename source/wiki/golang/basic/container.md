---
layout: wiki
wiki: golang
title: 容器类型
order: 64
---

|数据类型|非定义类型字面表示|直接部分值尺寸|字面量|零值|
|:--:|:--:|:--:|:--:|:--:|
|数组|[EleNum]EleType|EleSize\*EleNum|[EleNum]EleType{ele1, ele2, ...}|`[EleNum]EleType{}` => `[EleNum]EleType{EleZero1, EleZero2, ...}`|
|切片|[]EleType|3 words(编译器实现)|[]EleType{ele1, ele2, ...}|`nil` => `[]EleType(nil)`|
|映射|map[KeyEleType]ValEleType|1 word(编译器实现)|map[KeyEleType]ValEleType{KeyEle1: ValEle1, ...}|`nil` => `map[KeyEleType]ValEleType(nil)`|

> * `[]EleType{}` 是空切片，不同于 `[]EleType(nil)` 是 nil 切片，同理 `map[KeyEleType]ValEleType{}`
> * 字面量不可寻址但可以取地址 `a := &[]int{1, 2, 3}`

## 数组 array

* 使用数组指针在 `for range` 时减少复制
* 数组可以作为切片的底层类型，所以可以由数组生成切片
* `len` 和 `cap` 编译时估值，可以看作常量

## 切片 slice
```go
type _slice struct {
    elements unsafe.Pointer
    len int
    cap int
}
```

### 创建
* 字面量赋值
* new(slice): 返回地址，但是 new 的值都是零值，所以没意义
* make(slice, len, cap): 空切片

### 操作
* 索引: `slice[i]`
* `slice[start:end:max]`
* `append`
* `copy`
* `len` 和 `cap`
* `slice...`: 展开 slice，作为可变参数的输入
* 遍历
* 反射修改切片长度和容量，效率低于直接赋值修改，尽量不用

## 映射 map
```go
type _hashmap *hashtableImp
```
映射元素不可寻址

### 创建
* 字面量赋值
* `new(map)`: 返回地址，但是 new 的值都是零值，所以没意义
* `make(map, len)`: 空映射

### 操作
* 索引: `map[k]`
* `copy`
* 删除: `delete(map, key)` 添加: `map[key] = val`
* 遍历: 遍历的同时对映射进行修改时
    * 删除未遍历的条目 A，则 A 绝对不会被遍历到
    * 增加未遍历的条目 B，不保证 B 会被遍历到
    > 拉链法创建的 hashmap，如果增加的 B 的 hash 在表中 hash 值已经被遍历过，那么就不会观察到这个值
* 模拟集合: map[int]struct{}
