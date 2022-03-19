# `async`与`await`

rust 语言标准中提供 `async` 与 `await` 关键词
这里简单介绍各自的使用

- `await`
  await 的使用相对比较简单：  
   在实现了 `Future` 的 `trait` 的类型实例后面
  且在 `async` 包裹的代码块中，使用 `.await`
  表示将当前协程在这里挂起等待 `Future` 完成并返回值

  ```rust
  async{
    // 假设fut 是个 实现 `Future` 的变量
    let fut = ...

    let result = fut.await;
  }
  ```

  目前`await`仅有这样用法

- `async`

  1. `async` 可以作为函数签名的一部分修饰一个函数
     `async` 将会使得函数返回一个 `impl Future`的对象。这个对象是由编译器
     生成的

     考虑如下代码

     ```rust
     async fn async_add(i:u32, b:u32)->u32{
         i + b
     }
     ```

     编译器可能会产生一个这样的函数

     ```rust
     fn async_add(i:u32, b:u32)->impl Future<Output=u32>{
         async move{
             i + b
         }
     }
     ```

  2. `async` 直接修饰代码块，代码块可作为表达式使用

     ```rust
     async {
         // do somethings
     };
     ```

     有些情况，要求 async 代码块产生的`Future` 拥有 `'static`生命周期
     那么就要捕获代码块外的引用变量所有权，与闭包可以使用类似的写法

     ```rust
     async move{
       // do somethings
     };
     ```

  3. `async` 与 闭包
     在部分的闭包参数中，对于闭包的约束条件可能如下

     ```rust
     async fn async_handle<F, Fut>(func: F)
     where
        F : FnOnce()->Fut,
        Fut : Future<Output = u32>{
            ...
        }
     ```

     显然 这里的 `func` 是一个 async 的闭包，但是，如果直接使用如下代码

     ```rust
     async || 12u32
     ```

     将会收到以下编译器报错

     ```bash
     error[E0658]: async closures are unstable
     ```

     这里就可以考虑跟环一种书写方式
     让闭包返回一个 `async` 代码块

     ```rust
     || async { 12u32 }
     ```

     这样就能通过编译了
