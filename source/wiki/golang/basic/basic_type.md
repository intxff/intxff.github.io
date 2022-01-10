---
layout: wiki
wiki: golang
title: 内置基本类型
order: 63
---

## 布尔
* 值尺寸: 1 byte
* 字面量: `true` `false`
* 零值: `false`

## 数值型

|类型|值尺寸|零值|字面量|
|:--:|:--:|:--:|:--:|
|int8, uint8(byte)|1 bytes|0|相应范围的整型字面量，byte 的字面量可以是个 character|
|int16, uint16|2 bytes|0|相应范围的整型字面量|
|int32(rune), uint32|4 bytes|0|相应范围的整型字面量，rune 的字面量可以是个 unicode 码点|
|int64, uint64|8 bytes|0|相应范围的整型字面量|
|int, uint|取决于编译器实现，通常为 1 word|0|相应范围的整型字面量|
|uinptr|取决于编译器实现，要能容纳所有地址|0|相应范围的整型字面量|
|float32|4 bytes|0.0|相应范围的浮点字面量|
|float64|8 bytes|0.0|相应范围的浮点字面量|
|complex64|8 bytes|(0+0i)|float32 实部和虚部|
|complex128|16 bytes|(0+0i)|float64 实部和虚部|

### 字面量
* 整数型: 二进制 `0(b|B)` 八进制 `0[o|O]` 十六进制 `0(x|X)`
* 浮点型: 十进制浮点数(底数为 10 ，表示为 `e`) 十六进制浮点数(底数为 2 ，表示为 `p`)
* rune: 八进制 `\` 十六进制 `\x` 码点编码`\(u|U)`
```go
0b10110 0123 0x123 123
1.23e+10 1.23e-10 0x1.23p10 0x1.23p-10
```
字面量作为值是无名常量，具有默认类型
* 整型字面量默认类型是 int
* 浮点型字面量默认类型是 float64

## 字符串

```go
type StringHeader struct {
    Data uintptr // 指向 []byte
    Len int
}
// 而切片的结构是
type SliceHeader struct {
    Data uintptr
    Len int
    Cap int
}
// 所以 string 可以看成 readonly slice
```

字符串是按照 utf-8 编码存储

### 值尺寸
直接部分至少为 2 words，但是 go specification 未作要求，取决于编译器实现。间接部分根据存储内容决定。

### 零值
空字符串 ""

### 字面量
* 单行字符串: `""`
* 多行字符串: \`\`

### 相关操作

* 求长度: `len(string)` 返回的是 utf-8 编码的字节数
* 遍历
    * 字节遍历
    ```go
    for k, v := range []byte(string){
        fmt.Printf("seq: %v, value: %c\n", k, v)
    }
    ```
    * 码点遍历
    ```go
    for k, v := range string {
        fmt.Printf("seq: %v, value: %c\n", k, v)
    }
    // 上面这个的 seq 是字节序，下面这个是码点序。
    // 也就是说，如果字符串不转换成 []rune，读取是按照 utf-8 读取输出的
    for k, v := range []rune(string) {
        fmt.Printf("seq: %v, value: %c\n", k, v)
    }
    ```
* 转换: []byte string 和 []rune 之间的转换，由于 string 只读，string 与另两者之间的互转都是深复制
    * []byte <-> []rune 如果用 string 做桥梁，有 2 次深复制，效率低，不用
    * []byte <-> []rune 使用 unicode/utf8 库
    * []byte -> []rune 使用 byte 库，byte 库中没有 []rune -> []byte
    * string <-> []byte 有编译器优化不需要深复制的情形
        * for range string -> []byte
        * map[key] **读取(写入时无效)** key 对应的值时 []byte -> string
        * 字符串比较时 []byte -> string
        * 字符串衔接(至少有一个非空字符串参与) []byte -> string 
        > `s := " " + string([]byte) + string([]byte)`// 1 次深复制
        > `s := string([]byte) + string([]byte)` // 3 次深复制
        > 有非空字符串参与，也许所以可以先分配非空字符串堆上内存块，之后将 []byte 的堆上地址指定过去？
* 拼接
    * \+ 运算: 大量拼接效率低，因为要不断申请复制内存
    * strings.Builder，bytes.Buffer，[]byte: 效率差不多，减少内存分配次数才能更高，所以如果确定拼接后长度，预申请足够长度的 []byte 会是最高的效率

* 比较
   对于 `== !=` 的比较，如果两字符串的底层指针指向同一位置，那么比较长度就可知道，复杂度 O(1)；如果不同，那么就要比较长度再比较字节，复杂度是 O(n)。 
   所以字符串比较应尽量做相同底层指针的比较，也就是字符串赋值要尽量共享底层元素
