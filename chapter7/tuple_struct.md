# 元组结构体

有些情况下，我们比元组含有更多信息的数据类型，但是有不必指定每个 field 名称（毕竟给变量取名可是痛苦面具之一），比如颜色信息 RGB(11,22,33) 或者 Pos(1,2)等
这时候，就可以考虑元组结构体了
元组结构体，其类似于一个有名字的元组

## 声明

```rust
struct Name(u32, String, usize);
```

> 注意，元组结构体末尾必须以`;`结尾

与 Named 结构体类似，元组结构体可以单独控制每个 field 的可见性

```rust
pub struct Name(pub u32, String, usize);
```

## 创建

元组结构体创建类似于函数调用
将每个阐述按顺序传递

```rust
let b = Name(12, "iif".into(), 4);
```

## 变量访问

元组变量访问方式与元组一致
但是需要访问 field 的可见权限

```rust
let b = Name(12, "iif".into(), 4);
let c : String = b.1
```
