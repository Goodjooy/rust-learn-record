# Named 结构体

最经典的结构体格式,与 C 语言的格式十分相似

## 声明

以下是一个可能的结构体

```rust
struct S{
    a:i8,
    b:String
}
```

通过 `struct` 关键词标识，然后是结构体名称
紧跟一对`{}`用于添加结构体内部的变量

对于结构体中的每一个 field 都可以单独控制访问权限
与结构体访问权限相互独立

例如

```rust
pub struct S{
    pub a:i8,
    b:String
}
```

中，`a`将对外部可见，而`b` 只对当前的模块可见

## 创建结构体

当结构体中全部 field 在当前上下文中是可见的话，可以使用以下代码创建结构体

```rust
let b = S{ a:11, b:"abc".into() };
```

如果当前代码块中有与结构体中 field 同名的变量，可以省略 field 名称

```rust
let a = 11;
let b = String::from("abc");
let s = S{a, b};
```

如果新创建的结构体变量与原有的某个变量只在部分 field 有所不同，可以使用以下语句创建

```rust
let b = S{ a:11, b:"abc".into() };
let c = S{ a: 12, ..b };
```

## 变量访问

Named 结构体通过 `<val>.<name>` 来访问内部变量

> 访问时必须拥有访问权限

```rust
let b = S{ a:11, b:"abc".into() };
let c = b.a;
```
