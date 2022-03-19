# 发明一个`Future`[^1]

假设我们需要一个 `Future` 的对象，那么需要些什么？

- 也许需要提供一个函数，用来给异步运行时来判断任务状态
- 也许需要在函数参数提供一个回调函数，在任务完成后告知异步运行时

基于以上 2 点，我们制作出了一个简单的`trait`,由于其与标准库中提供的`Future`更加简单，这里称作 `SimpleFuture`

```rust
pub trait SimpleFuture{
    type Output;
    fn poll(&mut self,wake:fn())->Poll<Self::Output>;
}

pub enum Poll<T>{
    Pending,
    Ready(T)
}
```

`SimpleFuture` 提供了一个 `poll` 方法以给异步运行时调用，这个函数将会促使这个 `Future` 开始执行任务

> 值得注意的是，rust 的异步是完全惰性的，在不使用 await 或者 `spawn` 独立运行协程时，协程任务不会被执行

- 如果任务完成了，`poll`将会返回 `Poll::Ready(T)` 来告知运行时异步任务完成
- 如果任务还无法完成,`poll` 将会返回 `Poll::Pending` 来告知运行时异步任务需要挂起。当异步任务完成后，将会调用 `wake` 来告知运行时任务完成，需要再次调用 `poll` 来获取异步的结果(也叫做唤醒)

现在，假设我们拥有一个异步等待的`Socket`，我们为它实现 `SimpleFuture`

```rust
struct AsyncSocketRead<'socket>{
    socket: & 'socket Socket
}

impl<'s> SimpleFuture for AsyncSocketRead<'s>{
    type Output = Vec<u8>;

    fn poll(&mut self, wake : fn())->Poll<Self::Output>{
        if self.socket.has_data_to_read(){
            // 套接字中有可读取的数据，返回读取完成的数据
            Poll::Ready(self.socket.read_buf())
        }else{
            // 这个套接字目前没有任何的数据可以读取
            //
            // 程序将wake 传递给当套接字有可读取数据时的回调函数
            // 此时将会被运行时再次调用，以获取最新的数据
            self.set_readable_callback(wake);
            // 由于需要等待，因此返回 Poll::Pending
            Poll::Pending
        }
    }
}
```

同时，可以很容易发现，`SimpleFuture` 可以轻松实现嵌套的异步结构

```rust
struct AndThenFut<F, S>{
    first: Option<F>,
    second: S
}

impl<F, S> SimpleFuture for AndThenFut<F, S>
where
    F : SimpleFuture<Output = ()>,
    S : SimpleFuture<Output = ()>,
{
    type Output = ()

    fn poll(&mut self, wake : fn())->Poll<Self::Output>{
        if let Some(f_fut) = &mut self.first{
            match f_fut.poll(wake){
                // 先执行first 异步任务，如果任务完成，就移除这个任务
                Poll::Ready(())=> {
                    self.first.take();
                },
                // 否则继续等待
                Poll::Pending => return Poll::Pending
            };
        }
        // 执行second 任务
        self.second.poll(wake)
    }
}

```

这些看起来似乎不错，`SimpleFuture` 可以反应出异步运行时的控制流过程，同时可以一定程度避免了回调地狱。是时候看看标准库中的`Future`[^2] 了

```rust
trait Future {
    type Output;
    fn poll(
        // 这里的self类型从 `&mut Self` 变成了 `Pin<&mut Self>`
        self: Pin<&mut Self>,
        // 这里的唤醒函数从 `wake: fn()` to `cx: &mut Context<'_>`:
        cx: &mut Context<'_>,
    ) -> Poll<Self::Output>;
}
```

注意到 2 点变化

1. `self` 的类型从 `&mut Self` 变换为 `Pin<&mut Self>`
    这种变换将在后续提及，采用这种方案是为了能够构造型如

    ```rust
    async{
        let a = [1,2,3,4];
        async_do(&a);
    }
    ```

    这样的**自引用**的情况

2. `wake` 从一个简单的函数指针 `fn()` 变换为 `&mut Context<'_>`
    `fn()` 只是一个简单的函数指针，无法储存更多的关于异步任务的更多信息，
    而`Context` 将可以告知运行时哪个异步任务唤醒了，并执行相应的操作。

    `Context` 还可以提供 `Waker` 来支持更加复杂的唤醒行为

[^1]: https://rust-lang.github.io/async-book/02_execution/02_future.html
[^2]: https://doc.rust-lang.org/std/future/trait.Future.html
