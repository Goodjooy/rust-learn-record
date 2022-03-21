# 控制流：循环

- `loop` 循环
  死循环
  最简单的循环，除非使用`break` 或者其他退出方式，否则循环不会退出
  在break语句之后可以携带一个值，这个值将会作为整个loop语句的值，
  在 [表达式与语句](../chapter4/readme.md) 中将会更多地进行讨论

  ```rust
      loop{
          // do something
          if a==11{
              break 11;
          }
      }
  ```

- `while` 循环
  while 循环与其他语言的格式基本一致

  ```rust
    while a < 10{
        // do something
        a+=1;
    }
  ```

  > 在[模式匹配]()中将会介绍 `while let` 语句

- `for` 循环
  for 循环与 Python 中的格式相似

  ```rust
  let a = [1,2,3,4,5];
  for i in a{
      // do something
  }
  ```
