# 第一公民：函数

通常，一个函数长这样

```rust
    fn <function>(<args>)-><return>{
        // do something
    }
```

- `<function>` 为函数名称，将在调用时使用

  ```rust
      fn add(l:u32,r:u32)->u32{
          l + r
      }

      let c = add(1,3);
  ```

- `<args>` 为函数的参数列表，格式为 `<name>:<type>` 使用`,`分割多各参数
  例如`a:u32`,`b:String`等
- `<return>` 为函数返回值，如果无返回值可以缺省

  > 函数返回，既可以 使用函数最后一个表达式的值作为返回值，也可以用`return`语句来返回

  ```rust
      fn add(l:u32,r:u32,o:&mut u32){
          *o=l+y
      }
  ```

  > - `&mut u32` 是一个 对 u32 的可变引用，可变与不可变将在[变量](../chapter6/readme.md)中介绍；引用类型将在 [所有权、引用与生命周期](../chapter10/readme.md) 中详细介绍

  > - `*o` 是对 `&mut u32` 类型的解引用操作，将会在 [智能指针与 Deref]() 部分被再次提及

在 `fn` 关键词前，可以添加额外的关键词

`<vis> <async|...> fn ...`

vis 是标记函数的访问性， 在 vis 与 fn 之间的是额外的修饰。比如 `async` 修饰就是标记该函数为异步函数，`const` 就是标记该函数可以在编译期确定其值等。

```rust
pub async fn add_async(a:u32,b:u32,duration:Duration)->u32{
    sleep(duration).await;
    return a + b;
}
```

这个函数就是一个异步函数
> `pub` 为可见性控制关键词，将于 [访问权限控制](../chapter9/readme.md) 中主要介绍
> `async` 与 `await` 关键字将在 [Rust async 编程](../chapter25/readme.md) 进一步介绍
