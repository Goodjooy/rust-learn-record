# 所有权与所有权转移

假设我们有以下代码

```rust
let a = 11u32;
let b = a;

println!("a is {}, b is {}", a, b);
```

这段代码与其他语言几乎没有什么两样

自然而然地，考虑将 `u32` 更换为 `String`

```rust
let a = String::from("owner");
let b = a;

println!("a is {}, b is {}", a, b);
```

似乎这段代码应该也没有问题  
但是，rust 报告了一个异常

```bash
error[E0382]: borrow of moved value: `a`
 --> src/bin/owner.rs:5:34
  |
2 |     let a = String::from("owner");
  |         - move occurs because `a` has type `String`, which does not implement the `Copy` trait
3 |     let b = a;
  |             - value moved here
4 |
5 |     println!("a is {}, b is {}", a, b);
  |                                  ^ value borrowed here after move
  |
```

## 所有权

在 rust 中，每个值都会与变量唯一绑定,这种绑定关系就是所有权。

- 唯一性：同一时间一个值只能与一个变量绑定，即只有一个变量拥有一个值的所有权。

例如 `let a = 11u32;` 这一语句，就是将值 `11u32` 与 变量 `a` 唯一绑定，即 `a` 具有变量 `11u32`所有权

### 所有权转移

在一定条件下，会发生所有权的移动。当所有权移动以后，原来持有所有权的变量将不可用。

- 赋值
- 传递函数参数
- 从函数中返回

#### 赋值语句

```rust
let a = String::from("owner");
let b = a;
```

以上代码中，在第一行 `a` 拥有 `String` 值的所有权，在第二行中 `let b = a;` 将 `a` 中值赋给了 `b`。因此发生了所有权转移。

#### 作为参数传递

考虑有以下函数

```rust
fn str_size(s: String) -> usize {
    s.len()
}
```

函数参数列表中的 `s: String` 表明这个函数将会获取传递进来的`String`的所有权。当调用该函数后，原有的`String`将不再可用

#### 从函数参数中返回

如果调用函数后仍然想要继续使用数据，一种可行的方案将是将获取所有权的参数再次返回

```rust
fn str_size(s: String) -> (usize, String) {
   (s.len(), s)
}
```

这样，在调用时就可以这样使用

```rust
fn main() {
    let a = String::from("owner");
    let (size, a): (usize, String) = str_size(a);
    println!("a is {} size {}", a, size);
}
```

不过，如此的操作略显繁琐。

## 所有权与变量内存释放

```rust
fn main() {
    let a =11u32;
    let b = 23i32;
}
```

显然，`a` 与 `b` 的值都是栈上变量，且拥有各自的所有权。当变量离开作用域时，拥有所有权的变量将会进行内存空间释放。

```rust
fn main() {
    let a =11u32;
    let b = 23i32;

    // b 被先释放空间
    // a 在之后释放空间
}
```

所有权机制保证了同一时间对于同一值有唯一的变量与其绑定。那么在释放内存时只要将持有所有权的变量的值进行释放就能保证不会出现内存泄漏和`Double Free`

考虑以下代码

```rust
struct Unit;

impl Drop for Unit {
    fn drop(&mut self) {
        println!("Now Unit Drop at 0x{:x}", self as *const Unit as usize)
    }
}

struct Unit2;

impl Drop for Unit2 {
    fn drop(&mut self) {
        println!("Now Unit2 Drop at 0x{:x}", self as *const Unit2 as usize)
    }
}
```

我们创建了 2 个空白结构体 `Unit` 以及 `Unit2` 并且手动实现了 `Drop` ,在释放时进行信息输出

> `std::ops::Drop` 是用户定义的数据结构析构过程，通常情况下编译器会自动生成析构代码，用户定义析构过程将可以补充自动产生的代码
> `impl XXX for XXXX` 将在 [impl 代码块]() 中进一步介绍

接下来分别创建 2 个实例

```rust
fn main() {
    let a = Unit;
    let b = Unit2;
}
```

然后就得到了以下输出

```bash
Now Unit2 Drop at 0xddfc53f4c0
Now Unit Drop at 0xddfc53f4b8
```

但是很多情况下，代码不会这么简单

比如
当出现了函数调用

```rust
fn foo(a: Unit){
    ...
}

fn main() {
    let a = Unit;
    let b = Unit2;

    foo(a)
}
```

会发现析构顺序发生了改变

```bash
Now Unit Drop at 0x379d5bf7e0
Now Unit2 Drop at 0x379d5bf810
```

当函数调用获取了变量的所有权，如果那个变量没有被作为返回值或者返回值的一部分返回，那么这个变量就会在函数结束后析构。  
上述代码中，由于 `foo` 函数获得了 `a` 的所有权。当 `foo` 结束时，参数列表的变量由于没有作为返回值返回，所以在函数退出时析构了。表现出来的现象就是 `Unit` 先于 `Unit2` 析构

可以明显注意到，利用所有权机制，可以保证内存释放的安全
