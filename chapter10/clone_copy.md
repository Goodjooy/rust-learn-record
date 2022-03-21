# 克隆（`Clone`）与拷贝（`Copy`）

## 克隆（`Clone`）

在之前的代码中，如果要在一个获得变量所有权的函数调用后让原来的变量依然可用，就需要将变量的值返回。这种做法相对繁琐，我们可以考虑传递原有值的副本。

```rust
fn main() {
    let a = String::from("123456");

    let size = foo(a.clone());

    println!("a is {}, len {}", a, size);
}

fn foo(s: String) -> usize {
    s.len()
}
```

以上代码可以成功运行，在函数调用时，我们使用 `a.clone()` 来获取一个 `a` 的副本。这个副本是对原来值的完全拷贝

> - `Clone::clone()` 是一个 `trait` , 用于拷贝数据（类似于 cpp 的拷贝构造函数）
> - 在 rust 中，没有所谓深拷贝与浅拷贝的区分。所有的拷贝都是深拷贝（区分深拷贝与浅拷贝某种意义上是一种错误）

rust 许多内部提供的数据类型都实现了 `Clone`

---

有些时候，我们希望用户定义的结构也可以进行 `clone` 操作。在大多数情况下，我们可以直接使用驱动宏提供自动实现

```rust
#[derive(Clone)]
struct MyObj {
    a: u32,
    b: u32,
    c: i128,
}
```

这样，就可以对 `MyObj` 使用 `Clone::clone()` 来获取一个独立副本

> - `std::clone::Clone` 是一个`trait` , 将在 [泛型、trait]() 进一步讨论  
> - `#[derive(Clone)]` 是一个 `derive macro` 调用，将在 [宏]() 中进一步讨论

## 拷贝（`Copy`）[^1]

也许有人注意到
当数据类型为 `u32` 等类型时

```rust
    let a = 3u32;
    let b = a; // 所有权转移

    println!("a {} b{}", a, b);
```

即使在 `let b = a` 这行中发生了所有权转移，但是转移后的 `a` 依旧可用。

而当数据类型为 `String` 等类型时

```rust
    let a = String::from("string");
    let b = a;
    println!("b {}", b);
```

所有权转移后却让 `a` 不再可用

这是由于 `u32` 类型实现了`Copy` 而 `String` 没有实现 `Copy`

---

在发生所有权转移时，默认情况下使用的是移动语义。在移动语义情况下，当发生所有权转移时。原来持有所有权的变量将不可用。所有没有实现 `std::marker::Copy` 的类型都会在所有权转移时使用移动语义。`String` 就是典型的没有实现 `Copy` 的数据类型。

当一个类型实现了 `std::marker::Copy` ，在进行所有权转移时将使用拷贝语义。也就是说，这种类型的数据在发生所有权转移时将会进行拷贝，让获得所有权的变量获得原变量的值的副本，并且保持原有值可用,相当于每次发生所有权转移时都会自动进行`Clone`操作。基本数据类型都有实现 `std::marker::Copy`。 `u32` 就实现了`Copy`

### 对用户定义类型实现 `std::marker::Copy`

通常情况下，同样可以使用 `Derive Macro` 来自动完成 `Copy` 的实现，不过由于 `Copy` 是 `Clone` 的 `SupTrait` 所以在实现 `Copy` 时同时需要 `Clone` 实现

```rust
#[derive(Clone, Copy)]
struct MyObj {
    a: u32,
    b: u32,
    c: i128,
}
```

[^1]: https://doc.rust-lang.org/std/marker/trait.Copy.html
