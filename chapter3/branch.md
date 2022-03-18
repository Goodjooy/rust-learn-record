# 控制流：条件分支

与其他大部分语言一致，rust 提供`if` 与 `else` 关键词来提供条件分支

比如，一个可能的条件语句如下

```rust
    if a == 11 {
        // do something
    }
```

rust 也支持`else if` 语句

```rust
    if a==11{
        // do something
    }else if a==12{
        // do something
    }else{
        // do something
    }

```

以上语句也可以使用 `match` 替换

```rust
    match a{
        11 =>{
            // do something
        },
        12=>{
            // do something
        },
        _=>{
            // do something
        }
    }
```

> 在[模式匹配]()中将会介绍更多关于`match` 和另一种 `if let` 语句
> 在[表达式与语句](./chapter4/mod.md) 中将提及将 `if` 语句整体作为一个表达式使用的情况
