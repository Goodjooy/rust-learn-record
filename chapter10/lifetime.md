# 生命周期

对于每一个变量，rust 都会追踪其的生命周期

当一个变量离开了其所在的作用域，也就是生命周期结束。将会触发析构

```rust
{
    let a = 11;

}
// a 离开了作用域，生命周期结束，析构
```

对于拥有所有权的变量来说，生命周期与自身从创建到离开作用域析构的时间保持一致
但是对于引用变量而言却略有不同

```rust
let b = {
    let a = 11;
    b = &a;
    b
};
```

在意识模糊的情况下可能会写出以上的代码。 简单分析后我们发现：

- 变量 `a` 持有 `11`的所有权，在离开作用域后被析构
- 变量 `b` 获得了一个 `a` 的不可变引用，然后作为表达式的值返回

但是，当外部的 `b` 取得表达式的值时，引用指向的变量已经被析构了，此时 `b` 就是个**悬空指针**。
不过，以上代码 rust 不会允许通过编译，我们将获得以下错误输出

```bash
error[E0597]: `a` does not live long enough
 --> src/bin/owner.rs:4:17
  |
2 |     let b = {
  |         - borrow later stored here
3 |         let a = 11;
4 |         let b = &a;
  |                 ^^ borrowed value does not live long enough
5 |         b
6 |     };
  |     - `a` dropped here while still borrowed
```

原因也非常简单，`b` 为 `a` 的一个引用，引用的生命周期不能比原始值的生命周期更长。

在大多数情况下，编译器会自行判断生命周期。但是总有编译器判断不出来的情况
考虑如下代码

```rust
fn foo(a: &u32, b: &u32) -> &u32 {
    if a < b {
        a
    } else {
        b
    }
}
```

这段代码 rust 不会允许编译通过。因为函数参数中 `a` 和 `b` 分别是 2 个可能生命周期不同的引用，而返回值的引用不知道将会使用哪一个的生命周期。

## 生命周期标记

生命周期标记格式 为一个`'`开头后接一个合法标识符名称。  
`'r`.`'doc` 都是合法的生命周期标记

我们得到以下编译错误

```bash
error[E0106]: missing lifetime specifier
 --> src/bin/owner.rs:3:29
  |
3 | fn foo(a: &u32, b: &u32) -> &u32 {
  |           ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `a` or `b`
help: consider introducing a named lifetime parameter
  |
3 | fn foo<'a>(a: &'a u32, b: &'a u32) -> &'a u32 {
  |       ++++     ++          ++          ++
```

编译器提示我们显示提供生命周期标记。并给出了简单的帮助.
根据提示修改代码后，就能通过编译了

```rust
fn foo<'a>(a: &'a u32, b: &'a u32) -> &'a u32 {
    if a < b {
        a
    } else {
        b
    }
}
```

在另一种情况下，当结构体或者枚举中需要存放指向其他变量的引用时，也需要显示提供生命周期标记

```rust
struct MyObj<'a,'b> {
    a: & 'a u32,
    b: & 'b u32,
    c: i128,
}
```

在有些情况下，我们可以让编译器自动推导结构体的生命周期,那么就可以用 `'_` 来让编译器自动推导

```rust
fn foo2(a:MyObj<'_,'_>){
    ...
}
```

某些情况中，当使用高阶约束([Higher-ranked trait bounds])时，将会使用 `for<'r>`格式的泛型约束

考虑这个函数

```rust
fn foo3<F>(f: F)
where
    F: for<'a> FnOnce(&'a u32),
{
    let inner = 11;
    f(&inner);
}
```

注意到生命周期 `'a` 只与 `F` 泛型参数有关，因此可以将显示生命周期标记只限制在泛型约束中

> - `<F>` 是泛型参数格式，将在 [泛型、trait]中进一步讨论
> - `where` 语句是泛型参数的约束条件，将在 [泛型、trait] 中进一步讨论

生命周期的约束不但可以作用在生命周期上，也可以作用在泛型中

- `T : 'b` 表示 `T` 的生命周期至少与 `'b` 一样长
- `'a: 'b` 表示 `'a`标记的生命周期至少与 `b`一样长(存疑)

在生命周期标记中，有一种特殊的生命周期 : `'static`
这种生命周期是 rust 程序中最长的生命周期（甚至比`main`中变量生命周期还长）

- 字符串字面量生命周期为 `'static`
- `const`和 `static` 修饰的全局变量的引用是 `'static`,以下代码可以运行

  ```rust
  static A: u32 = 1;
  const B: u32 = 2;
  fn main() {
      let _r = foo(&A, &B);
  }

  fn foo<'a>(a: &'static u32, b: &'static u32) -> &'a u32
  where
      'static: 'a,
  {
      if a < b {
          a
      } else {
          b
      }
  }
  ```

- 通过 `Box::leak()` 方法获得的引用是 `'static` 的，这表明这个引用在之后的程序运行周期中将一直存在.
  以下代码将也能通过编译

  ```rust
  fn main() {
      let a = Box::new(1u32);
      let leak_a = Box::leak(a) as &u32;
      let b = Box::new(2u32);
      let leak_b = Box::leak(b) as &u32;
      let _r = foo(leak_a, leak_b);
  }

  fn foo<'a>(a: &'static u32, b: &'static u32) -> &'a u32
  where
      'static: 'a,
  {
      if a < b {
          a
      } else {
          b
      }
  }
  ```

> `Box` 是最简单的指向堆的智能指针，将在 [智能指针与 Deref]中进一步介绍

[higher-ranked trait bounds]: https://doc.rust-lang.org/reference/trait-bounds.html#higher-ranked-trait-bounds
[泛型、trait]: ../Readme.md
[智能指针与 deref]: ../Readme.md
