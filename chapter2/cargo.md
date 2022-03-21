# 使用 Cargo 构建项目和编译

打开命令行

```bash
cargo new demo
```

以上指令会在 ./demo 文件夹内新建一个 rust 项目，demo 为项目名称，可以任意指定。  
在 demo 文件夹内，文件夹格式如下

```bash
demo/
├── Cargo.toml
└── src
    └── main.rs
```

## `Cargo.toml` 文件

打开 Cargo.toml 看看

```toml
[package]
name = "demo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

这个文件是当前项目的配置文件，包括交叉编译配置、依赖配置、release 与 debug 模式行为配置等多种多样配置。  
具体可以[参考](https://doc.rust-lang.org/cargo/reference/manifest.html)

在`[package]` 中

- name 当前 crate 的名称
- version 当前 crate 的版本
- edition 当前 crate 的固化 rust 版本

> 在 rust 中，项目以`crate`为单位，在[代码模块化]()将会进一步说明

在 `[dependencies]` 中提供的是当前 crate 的依赖，  
由于目前项目无需任何依赖，所以胃口

> 在 [依赖管理]()中将更多讨论如何书写这部分配置文件

## `Src` 文件夹

src 文件夹是存放项目全部代码的地方，当前只有一个`main.rs`
后续所有的项目代码都将在 src 中

---

进入到`demo`文件夹，直接使用指令

```bash
cargo run
```

将看到以下输出

```bash
   Compiling demo v0.1.0 (/home/frozenstring/r/demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/demo`
Hello, world!
```

前 2 行，cargo 检查并编译了代码
后 2 行，cargo 执行了编译完成后的可执行文件

> cargo 有多种可用指令，可以通过 `cargo help`来获取更多信息
