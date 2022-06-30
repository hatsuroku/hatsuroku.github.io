---
layout: post
title: "Rust learning notes"
subtitle: "脑子不好的人的 Rust 学习记录"
date: 2022-06-27
author: "LXS"
header-style: text
tags:
  - "Rust"
---

# Rust 学习笔记

推荐：[Rust 语言圣经](https://course.rs/)

对中文母语者友好，语言幽默，章节安排合理。

我就看这个学的。



## Rust 的「所有权」

### 赋值等号

和 C++ 不同，Rust 的赋值等号默认是**移动语义**而不是拷贝语义，如果一个变量赋值给了另一个变量，则默认实现的是移动语义：

![image-20220627133320085](/img/in-post/2022-06-27-rust-learning-notes/image-20220627133320085.png)

只有在显式调用 `.clone()` 时，实现的才是复制内存的深拷贝。

但有一些类型的赋值等号默认实现的是拷贝语义：

![image-20220627133705697](/img/in-post/2022-06-27-rust-learning-notes/image-20220627133705697.png)

### 引用

Rust 中的引用其实更接近于其他语言中「弱引用」的概念：**借用值，但不影响该值的生命周期。**

引用默认是不可变的，如果要修改值则需要显示声明为`可变引用`：

```Rust
fn main() {
    let mut s = String::from("hello");
    // 传值的时候需要声明为 mut
    change(&mut s);
}
// 这里 some_string 的类型需要声明为 mut
fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```



#### 引用的作用域

和变量的作用域不一样，引用的作用域并不是由大括号决定的。

**一个引用的作用域从声明的地方开始一直持续到最后一次使用为止。**

> 官方说明：
>
> 编译器在作用域结束之前判断不再使用的引用的能力被称为 **非词法作用域生命周期**（*Non-Lexical Lifetimes*，简称 NLL）。你可以在 [The Edition Guide](https://doc.rust-lang.org/edition-guide/rust-2018/ownership-and-lifetimes/non-lexical-lifetimes.html) 中阅读更多关于它的信息。

```rust
fn main() {
  let mut s = String::from("hello");

  // r1 被创建
  let r1 = &s;
  println!("{},", r1);
  // r1 生命周期结束
  
  // r2 被创建
  let r2 = &s;;
  println!("{},,", r2);
  // r2 生命周期结束
}

```



#### 引用的限制

`可变引用` 在同一个作用域只能拥有一个，`不可变引用` 则没有这种限制。

```Rust
let mut s = String::from("hello");

{
  let r1 = &mut s;
} // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

let r2 = &mut s;
```

`不可变引用` 和 `可变引用` 不可同时持有。（很容易理解，如果可以同时持有那么不可变引用就没意义了）

```Rust
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题

println!("{}, {}, and {}", r1, r2, r3);
```

但 `可变引用` 可以出现在 `不可变引用` 生命周期结束之后，这样就不是同时持有了。

```rust
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
println!("{} and {}", r1, r2);
// 此位置之后 r1 和 r2 不再使用

let r3 = &mut s; // 没问题
println!("{}", r3);
```



#### for 循环中的 ＆ 的意思

其实是解构赋值：https://stackoverflow.com/questions/57339201/what-is-the-purpose-of-before-the-loop-variable

```rust
fn largest(list: &[i32]) -> i32 {
    println!("{:?}", list);
    let mut largest = list[0];
    // list 的类型是 &[i32]
    
    // for loop 里的第一次循环可以理解为 let &i = list[0]
    // list[0] 的类型是 &i32
    // 所以 i 的类型为 i32（被解构）
    for &i in list {
        if i > largest {
            largest = i;
        }
    }
    largest
}
```



