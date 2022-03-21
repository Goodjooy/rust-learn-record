# 我是谁，我在哪：Rust Async 编程[^async]

异步编程又是一个比较复杂的环节
相对于线程等部分，rust 标准库只提供了异步的部分`trait`和
`async` 、`await` 等，而具体 async runtime 可以由社区实现

## 目录

- [为什么需要异步](./why_async.md)
- [`async`与`await`](./async_and_await.md)
- [发明一个`Future`](./future.md)
- [为什么要`Pin<&mut self>`，什么是`Pin`](pin.md)
<!--
- [异步的迭代器：`Stream`](TODO)
-->

[^async]: https://rust-lang.github.io/async-book/
