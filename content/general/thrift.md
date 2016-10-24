+++
date = "2016-10-21T17:58:03+08:00"
title = "thrift"
isCJKLanguage = true
tags = [
  "thrift",
  "crossplatform",
]
draft = true
categories = [
  "General",
]

+++

### 基本类型

- `bool` true 或者 false
- `byte` 8位有符号整型
- `i16`  16位有符号整型
- `i32`  32位有符号整型
- `i64`  64位有符号整型
- `double` 64位浮点型
- `string` 以UTF-8编码的一段文本

### 特殊类型
`binary` 未编码的字节序列

### 结构体

结构体包括一系列**强类型的字段**，每个字段都有一个**唯一的标识符**。看起来很像C语言的结构体。

```thrift
struct example {
    1: i32 number = 10,
    2: i64 bigNumber,
    3: double decimals,
    4: string name = "thrfity"
} 
```

### 容器类型

- `list` 列表
- `set`  集合
- `map`  键值对

### 异常

继承自对应语言的异常

```thrift
exception InvalidOperation {
    1: i32 what,
    2: string why
}
```

### 服务

一个服务由若干个**命名的函数，每个函数有一个参数列表和一个返回值类型**。从语义上说，就像是定义了一个接口或一个纯虚抽象类。

```thrift
service StringCache {
    void set(1: i32 key, 2: string value),
    string get(1: i32 key) throws (1: KeyNotFound knf),
    void delete(1: i32 key)
}
```

