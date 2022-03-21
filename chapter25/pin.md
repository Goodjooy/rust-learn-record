# 为什么要`Pin<&mut self>`，什么是`Pin` [^pin]

在前文中，遇到了 `Pin<&mut Self>` 的 `self` 类型  
现在来看看为什么用`Pin`、如何使用 `Pin`  

## 为什么

当我们使用`async` 代码块时

```rust
async {
    fut1.await;
    fut2.await;
}
```

时，编译器会生成大概如下的代码

```rust
struct FutTask{
    fut1:FutOne,
    fut2:FutTwo,
    status:Status
}

enum Status{
    WaitFirst,
    WaitSecond,
    Ok
}

impl Future for FutTask{
    type Output = ()

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<()> {
        loop {
            match self.status {
                Status::WaitFirst => match self.fut1.poll(..) {
                    Poll::Ready(()) => self.status = Status::WaitSecond,
                    Poll::Pending => return Poll::Pending,
                }
                Status::WaitSecond => match self.fut2.poll(..) {
                    Poll::Ready(()) => self.state = Status::Ok,
                    Poll::Pending => return Poll::Pending,
                }
                State::Done => return Poll::Ready(()),
            }
        }
    }
}
```

这样看起来似乎一切正常，但是，如果使用了以下代码块

```rust
async {
    let buf = [0u8;32];
    read_async_buf(&mut buf).await;

    let s = Cow<_,str> = String::from_utf8_pretty(&buf);

    print!("{}",&s);
}
```

当构造结构体时，出现了问题

```rust
struct ReadAsyncBuf<'b>{
    // 读取过程中的buf的可变引用
    buf : & 'b mut [u8]
}
struct FutTask{
    buf:[u8;32],
    read_async_buf: ReadAsyncBuf<'lifetime>,
    ...
}
```

很明显注意到，无法确定 `read_async_buf` 变量的生命周期，我们首先不能为 `FutTask` 添加额外的生命周期参数，因为这个内部生命周期的长度和 `FutTask` 是一样长的，但是也不能是 `'static` 因为 `FutTask` 的生命周期是短于全局变量的。  
替代的方案就是采用一个 `*const [u8]` 来代替原有的引用，但是这又引出新的问题。

rust 并不保证一个变量所在的内存位置会保持不变，在所有权转移时或者其他情况中，变量在内存中的地址将可能发生改变。如果我们只是将 `*const [u8]` 简单指向 `buf` 变量的内存，当 FutTask 移动后，我们注意到，`buf`的内存地址已经改变了，但是 `*const [u8]` 中指向的地址却没有改变。这将会导致指针指向一个可能已经释放的内存，导致可能存在的问题或者未定义行为。

而为了解决这个问题，就出现了 `Pin` 和 `Unpin` trait

> 只有`pin` 包裹的对象是 `!Unpin` 的才会在内存中保持内存地址的不变性

## 是什么

让我们用一个简单的实例来说明 pin  

假设我们有一个以下的自引用的结构体

```rust
struct PinString {
    s: String,
    ptr: *const String,
}

impl PinString {
    fn new(s: &str) -> Self {
        let s = Self {
            s: s.into(),
            ptr: std::ptr::null(),
        };
        s
    }

    fn init(&mut self) {
        let ptr = &self.s as *const String;
        self.ptr = ptr;
    }

    fn get_s(&self) -> &str {
        &self.s
    }

    fn get_ptr(&self) -> &str {
        assert!(!self.ptr.is_null(), "内部指针为空指针，请初始化指针");
        unsafe { &*(self.ptr) }
    }

    fn show_addr(&self) {
        println!(
            "String At 0x{:x}, ptr at 0x{:x}",
            &self.s as *const String as usize, self.ptr as usize
        )
    }
}
```

接下来，简单创建 2 个对象,并且打印对象信息

```rust
fn main() {
    let mut s1 = PinString::new("test1");
    s1.init();
    let mut s2 = PinString::new("test2");
    s2.init();

    println!("S1 info s:{:?} , ptr: {:?}", s1.get_s(), s1.get_ptr());
    s1.show_addr();
    println!("S2 info s:{:?} , ptr: {:?}", s2.get_s(), s2.get_ptr());
    s2.show_addr();
}
```

然后，我们得到预期的输出

```bash
S1 info s:"test1" , ptr: "test1"
String At 0x780bd1f8a8, ptr at 0x780bd1f8a8
S2 info s:"test2" , ptr: "test2"
String At 0x780bd1f8c8, ptr at 0x780bd1f8c8
```

接下来，来点小破坏。我们直接使用 `std::mem::swap` 交换 `s1` 与 `s2`

```rust
fn main() {
    let mut s1 = PinString::new("test1");
    s1.init();
    let mut s2 = PinString::new("test2");
    s2.init();

    std::mem::swap(&mut s1, &mut s2);

    println!("S1 info s:{:?} , ptr: {:?}", s1.get_s(), s1.get_ptr());
    s1.show_addr();
    println!("S2 info s:{:?} , ptr: {:?}", s2.get_s(), s2.get_ptr());
    s2.show_addr();
}
```

然后，我们就得到了非预期的输出

```bash
S1 info s:"test2" , ptr: "test1"
String At 0xa18797f488, ptr at 0xa18797f4a8
S2 info s:"test1" , ptr: "test2"
String At 0xa18797f4a8, ptr at 0xa18797f488
```

我们发现 `s2` 中指针指向了 `s1`中的字符串，s1 的 指针同理。指针指向的对象移动了，但是我们指针却没有更新。  

此时，我们就需要 `Pin` 来固定变量在内存中的位置

## 如何使用

现在开始看看如何使用 `Pin` 吧！  
需要注意的是，如果要让`Pin<T>` 真正不会移动`T` 的内存地址，就要求 `T` 的类型为 `!Unpin` 也就是**可以被固定的**。而如果 `T` 的类型是 `Unpin` 表示`T` **不可被固定**，那么 `Pin<T>` 的行为和 `T` 会保持一致

rust 的大部分的内建数据类型以及大部分的用户定义类型都是 `Unpin` 的，这意味着我们即使使用 `Pin<T>` 包裹也不会有任何变化  

然而，有些情况，我们需要 `!Unpin` 来保证不会产生内存移动以避免可能存在的问题。  
通常情况下，我们可以在结构体内部引入 `PhantomPinned` field 来标记我们的结构体是`!Unpin` 的

### 固定在栈上

为了能在栈上固定的数据结构，对原来的数据结构进行了修改

```rust
struct PinString {
    s: String,
    ptr: *const String,
    _pin: PhantomPinned,  // 这个变量标记`PinString` 为 `!Unpin`
}

impl PinString {
    fn new(s: &str) -> Self {
        let s = Self {
            s: s.into(),
            ptr: std::ptr::null(),
            _pin: PhantomPinned,
        };
        s
    }

    fn init(self: Pin<&mut Self>) {
        let ptr = &self.s as *const String;
        let this = unsafe { self.get_unchecked_mut() };
        this.ptr = ptr;
    }

    fn get_s(self: Pin<&Self>) -> &str {
        &self.get_ref().s
    }

    fn get_ptr(self: Pin<&Self>) -> &str {
        assert!(!self.ptr.is_null(), "内部指针为空指针，请初始化指针");
        unsafe { &*(self.ptr) }
    }

    fn show_addr(self: Pin<&Self>) {
        println!(
            "String At 0x{:x}, ptr at 0x{:x}",
            &self.s as *const String as usize, self.ptr as usize
        )
    }
}
```

修改了`main` 中的代码，以使用 Pin 来将值固定在栈上

```rust
fn main() {
    let mut s1 = PinString::new("test1");
    // 将值固定在栈上是 `unsafe` 的行为
    // 使用 同名变量覆盖将原有变量覆盖
    let mut s1 = unsafe { Pin::new_unchecked(&mut s1) };
    PinString::init(s1.as_mut());
    let mut s2 = PinString::new("test2");
    let mut s2 = unsafe { Pin::new_unchecked(&mut s2) };
    PinString::init(s2.as_mut());

    println!(
        "S1 info s:{:?} , ptr: {:?}",
        PinString::get_s(s1.as_ref()),
        PinString::get_ptr(s1.as_ref())
    );
    PinString::show_addr(s1.as_ref());

    println!(
        "S2 info s:{:?} , ptr: {:?}",
        PinString::get_s(s2.as_ref()),
        PinString::get_ptr(s2.as_ref())
    );
    PinString::show_addr(s2.as_ref());
}
```

然后我们得到了我们的输出

```bash
S1 info s:"test1" , ptr: "test1"
String At 0x7235eff6d8, ptr at 0x7235eff6d8
S2 info s:"test2" , ptr: "test2"
String At 0x7235eff700, ptr at 0x7235eff700
```

接下来，我们试图交换 s1 与 s2 的内存位置
然而，类型系统阻止了我们的这种行为

```bash
error[E0277]: `PhantomPinned` cannot be unpinned
```

> 固定在栈上是 `unsafe`的 有些 crate 可以使用 `pin_mut!` 等宏简单在栈上固定数据

### 固定在堆上

将数据固定到堆上将使得我们的数据获得一个固定的内存地址。而固定在栈上的值只在其生命周期期间保持内存固定。  

我们将刚刚的代码进行再次修改，  
固定在堆上的数据相对没有复杂的代码

```rust
struct PinString {
    s: String,
    ptr: *const String,
    _pin: PhantomPinned,
}

impl PinString {
    fn new(s: &str) -> Pin<Box<Self>> {
        let s = Self {
            s: s.into(),
            ptr: std::ptr::null(),
            _pin: PhantomPinned,
        };
        // 这些代码将数据固定到堆上
        let mut boxed = Box::pin(s);
        let ptr = &boxed.as_ref().s as *const String;
        unsafe { boxed.as_mut().get_unchecked_mut().ptr = ptr };
        boxed
    }

    fn get_s(self: Pin<&Self>) -> &str {
        &self.get_ref().s
    }

    fn get_ptr(self: Pin<&Self>) -> &str {
        assert!(!self.ptr.is_null(), "内部指针为空指针，请初始化指针");
        unsafe { &*(self.ptr) }
    }

    fn show_addr(self: Pin<&Self>) {
        println!(
            "String At 0x{:x}, ptr at 0x{:x}",
            &self.s as *const String as usize, self.ptr as usize
        )
    }
}
```

修改后的`main` 如下

```rust
fn main() {
    let mut s1 = PinString::new("test1");
    let mut s2 = PinString::new("test2");

    println!(
        "S1 info s:{:?} , ptr: {:?}",
        PinString::get_s(s1.as_ref()),
        PinString::get_ptr(s1.as_ref())
    );
    PinString::show_addr(s1.as_ref());

    println!(
        "S2 info s:{:?} , ptr: {:?}",
        PinString::get_s(s2.as_ref()),
        PinString::get_ptr(s2.as_ref())
    );
    PinString::show_addr(s2.as_ref());

    std::mem::swap(&mut s1, &mut s2);

    println!();
    println!(
        "S1 info s:{:?} , ptr: {:?}",
        PinString::get_s(s1.as_ref()),
        PinString::get_ptr(s1.as_ref())
    );
    PinString::show_addr(s1.as_ref());

    println!(
        "S2 info s:{:?} , ptr: {:?}",
        PinString::get_s(s2.as_ref()),
        PinString::get_ptr(s2.as_ref())
    );
    PinString::show_addr(s2.as_ref());
}
```

我们得到输出

```bash
S1 info s:"test1" , ptr: "test1"
String At 0x1c7a6e8f4f0, ptr at 0x1c7a6e8f4f0
S2 info s:"test2" , ptr: "test2"
String At 0x1c7a6e8f5b0, ptr at 0x1c7a6e8f5b0

S1 info s:"test2" , ptr: "test2"
String At 0x1c7a6e8f5b0, ptr at 0x1c7a6e8f5b0
S2 info s:"test1" , ptr: "test1"
String At 0x1c7a6e8f4f0, ptr at 0x1c7a6e8f4f0
```

可以看到，虽然进行了内存交换，但是我们的指针仍然指向正确的位置

## 总结

1. 如果 `T` 是 `Unpin`（默认情况） 那么 `Pin<'a,T>` 的行为与 `&'a mut T`一致；  
   如果 `T` 是 `!Unpin` 那么 `Pin<T>` 将会固定数据在内存位置

2. 从 `Pinned T` 中获取 `&mut T` 当 `T:!Unpin` 时是`unsafe`的

3. 大多数的内置类型，和普通的用户定义类型都会实现 `Unpin`除了 `async/await`生成的 `impl Future`

4. 在 `nightly` 版本的 rust 中可以实现 `!Unpin`，在 `stable` 版本的 rust 中可以使用 `std::marker::PhantomPinned` 来将类型标记为 `!Unpin`

5. 数据可以固定在栈和堆上
   - 固定在栈上是 `unsafe` 的
   - 固定在堆是安全的，可以用 `Box::pin` 来便捷创建
6. 在 固定`T:!Unpin` 后，你需要维护数据固定的内存位置不会失效和变成它用在从固定开始到数据被释放

[^pin]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
