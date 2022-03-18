# 一切开始的开始：安装 rust 开发环境

## 安装`rustup`

### Linux 或者 MacOS

如果是在 Linux 或者 macOs  
打开一个命令行终端  
在终端输入以下指令

```bash
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

这条命令下载了一个用于下载安装最新版本的 `stable` 的 rust 的脚本
安装过程中可能需要输入密码

如果安装顺利，你将看到以下命令行输出

```bash
Rust is installed now. Great!
```

> 通常情况下，Linux 或者 macOs 都会有自带的 C 语言编译器，如果缺失，可能会导致 rust 无法链接。尝试安装 C 语言编译器以解决相关问题

### Windows

如果是 Windows 下安装 Rust
可以前往 [rust 安装](https://www.rust-lang.org/tools/install)
下载最新的安装包，然后进行 rust 的安装（一般情况下可以全部默认）

- 通过配置 环境变量 `RUSTUP_HOME` 和 `CARGO_HOME`可以改变 rustup 和 cargo 的安装目录

> 注意： 默认情况下，rust 会使用 `msvc` 链接器，请确保有安装 相关工具链

### 其他安装方式

参考：[Other Rust Installation Methods](https://forge.rust-lang.org/infra/other-installation-methods.html)

## 安装成功标志

具体版本可能有所不同

打开新的终端，输入

- 查看 rustc 编译器版本

  ```bash
  rustc --version
  ```

  然后将可以看到以下输出

  ```bash
  rustc 1.59.0 (9d1b2106e 2022-02-23)
  ```

- 查看 cargo(包管理工具) 版本

  ```bash
  cargo --version
  ```

  可以得到输出

  ```bash
  cargo 1.59.0 (49d8809dc 2022-02-10)
  ```

- 查看 rustup (rust 工具链安装器) 版本

  ```bash
  rustup --version

  ```

  可以得到输出

  ```bash
  rustup 1.24.3 (ce5817a94 2021-05-31)
  info: This is the version for the rustup toolchain manager, not the rustc compiler.
  info: The currently active `rustc` version is `rustc 1.59.0 (9d1b2106e 2022-02-23)`
  ```

自此，rust 相关开发环境安装完成
