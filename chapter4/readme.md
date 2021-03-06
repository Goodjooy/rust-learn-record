# 也许应该消灭一些中间变量：表达式与语句

在 rust 中，提供 表达式 与 语句 来实现高度灵活的代码实现

- 表达式
  表达式的形式非常自由，拥有多种多样的格式，通常情况下，表达式的最后一个表达将作为表达式的值,表达式的值可以作为函数返回值返回，或者赋值给变量
  表达式可以多级嵌套

  - 最基本的表达式，与其他语言类似。可以是函数调用，字面值，或者运算

    ```rust
            11+2*15 // 这是表达式
            call(11,2) // 这是表达式
            true // 这是表达式
    ```

  - 表达式可以是使用 `{` 和 `}` 包围起来的代码

    ```rust
        {
            let y = 11;
            12
        } // 这个整个代码块就是表达式，值为 12
    ```

  - 表达式可以是`if` 语句，如果`if` 作为表达式，那么要求 if 分支覆盖全部情况

    ```rust
        if a ==1 {
            true
        }else{
            false
        } // 这一整个代码块是表达式
    ```

  - 表达式可以是一个 `unsafe` 代码块

    > 这里示例代码并不需要 unsafe 包裹

    ```rust
            unsafe{
                let a = b as *const ();
                a
            }
    ```

  - 表达式可以是一个 match 代码块
    > match 作为表达式要求每个分支的返回类型一致

    ```rust
        match a{
            11=>{ 1u8 },
            _=>{ 0u8 }
        }
    ```

  - 表达式可以是一个带 break 的 loop

    ```rust
        loop{
            if a==11{
                break false;
            }
        } // 表达式的值类型为 bool
    ```

- 语句
    任何表达式赋值给一个变量，或者在表达式末尾追加 `;` 都会将表达式变为语句
    > 赋值语句记得在末尾追加`;`

    ```rust
        let a = 11 + 23; // 语句
        let c = if a == 54 { false } else { true };
    ```
