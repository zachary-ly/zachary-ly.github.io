+++
title = "rust 实现线性数据结构"
author = ["李伟"]
date = 2025-05-20T10:12:00+08:00
tags = ["rust"]
categories = ["数据结构"]
draft = false
IMAGE = "/images/pl-banner.jpg"
AUTHOR = "LW"
+++

## 线性数据结构 {#线性数据结构}

数组、栈、队列、双端队列数据项的顺序由添加或者删除的顺序决定


### 栈 - LIFO {#栈-lifo}

先进后出原则
根据栈的特性可以抽象以下数据操作：

-   new()
-   push(item)
-   pop()
-   peek()
-   is_empty()
-   size()


### RUST 实现栈 {#rust-实现栈}

```rust
// stack.rs

#[derive(Debug)]
struct Stack<T> {
    top: usize,
    data: Vec<T>
}

impl <T> Stack <T> {
    fn new() -> Self {
        Stack {
            top: 0,
            data: Vec::new()
        }
    }

    fn push(&mut self, val: T) {

        self.data.push(val);
        self.top += 1;
    }

    fn pop(&mut self) -> Option<T> {

        if self.top == 0 {return None;}
        self.top -= 1;
        // 使用Vec数据结构的pop()
        self.data.pop();
    }

    fn peek(&self) -> Option<T> {
        if self.top == 0 { return None; }
        self.data.get(self.top - 1);
    }

    fn is_empty(&self) -> usize {
        0 == self.top
    }

    fn size(&self) -> usize {
        self.top
    }
}
```
