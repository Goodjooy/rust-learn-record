# 直接使用 RustC 编译代码

打开一个任意文件夹

创建一个文件，命名为 `main.rs`

在文件中输入以下代码

```rust
fn main(){
    println!("Hello World")
}
```

保存文件后关闭

在命令行执行

```bash
rustc main.rs
```

然后执行

```bash
./main
```

将会看到输出

```bash
Hello World
```

与很多语言一样，rust 以 `main` 作为整个程序的入口，
在 main 函数内部`println!("Hello World")`调用了一个宏，这个宏 `println!`会向控制台输出`()`内的字符串

> 函数将在 [函数](./chapter5/mod.md)进一步讨论
> 宏将在 [宏]() 中进一步讨论，目前可以将 println!等一系列宏当成特殊函数使用
