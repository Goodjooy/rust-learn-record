# 你看不到我：访问权限控制

在 rust 中，有许多地方可以添加额外的可见性控制
默认情况下，都为 private

## 格式

pub 语句有 4 种格式

- `pub`
  最简单常用的 pub 模式，这种 pub 模式可以被父级一级一级向往传递
- `pub(crate)`
  只在当前的 crate 中可见，如果这个 crate 作为依赖被引入将不可被访问
- `pub(super)`
  只在父级可见
- `pub(in Path)`
  只在指定 path 中可见

## pub 的位置

- `static` 或者 `const` 变量前面

  ```rust
  pub static A : usize = 11;
  pub const B_CC : u8 = 13;
  ```

- 函数最前端

  ```rust
  pub fn method(){
      ...
  }
  ```

- 结构体本身或者 任意 field

  ```rust
  pub struct S{
      ...
      pub a : i32,
      b : &'static str,
      ...
  }
  ```

  > `& 'static str` 是静态声明周期的 str 引用，`'static`的生命周期标识这个引用的生命周期与程序一样长，在 [所有权、引用与生命周期]() 中将进一步讨论

- 枚举类型本身

  ```rust
  pub enum E{
      ...
  }
  ```

- trait 本身

  ```rust
  pub trait T{
      type A;
      ...
      fn func()->u32;
      ...
  }
  ```

  > 更多关于 `trait` 的内容将在 [泛型、trait]()进一步讨论
