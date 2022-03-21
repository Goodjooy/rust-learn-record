# 陌生的老朋友：Enum 类型

相对于其他语言的枚举类型，rust 提供了一个异常强大的枚举类型
配合模式匹配可以提供语义丰富的 rust 代码

## 声明

在枚举类型中，外部的包裹结构是一致的

```rust
enum EnumName{
    ...
}
```

对于枚举的每一个枚举类型，可以为以下类型
| 类型 |举例| 说明 |
| :---:| :--: |:----|
| Unit 类型 | `And`,`Or` | 与其他语言的枚举类似，只有枚举名 |
| Tuple 类型 | `Pos(usize,usize)` | 类似于内嵌元组结构体，除了枚举名外还有内部数据 |
| Named 类型 | `Pos{x:usize,y:usize}` | 类似于 Named 结构体 |

完整示例

```rust
enum EnumName{
    And,
    Or,
    Group(usize),
    Double{
        x:u32,
        u:u8
    }
}
```

对于枚举类型，可见性是统一的。enum 内部数据与 enum 可见性保持一致

```rust
pub enum EnumName{
    And,
    Or,
    Group(usize),
    Double{
        x:u32,
        u:u8
    }
}
```

整个枚举类型都对外可见

## 创建

创建枚举类型实例时，要指定使用哪一种 enum 枚举，
通过 `::` 来指定
例如

```rust
let and = EnumName::And;
let or = EnumName::Or;
let d = EnumName::Double{ x:1, y:1 };
```

## 内部数据获取

枚举类型由于有多种可能值，所以需要使用 `match` 或者 `if let` 来获取内部数据

```rust
let d = EnumName::Double{ x:1, y:1 };

match d{
    EnumName::And => {},
    EnumName::Or => {},
    EnumName::Double{x,y} => {},
    _=>{}
}
```

在`match` 语句内，使用 `_`来匹配其他全部模式

> `match` 将在 [模式匹配]()再次讨论
