+++
title = "使用 栈 实现括号匹配"
date = 2025-05-20T10:31:00+08:00
tags = ["rust"]
categories = ["数据结构"]
draft = false
IMAGE = "/images/pl-banner.jpg"
AUTHOR = "LW"
+++

{{< figure src="/ox-hugo/2025-05-20_10-36-49_screenshot.jpg" >}}

使用 栈数据结构 实现括号匹配的功能


## rust 实现 {#rust-实现}

-   将所有符号依次存储到Vec
-   使用 `balance` 判断是否平衡
-   '(' 入栈 否则 出栈
-   `Stack`.is_empty &amp;&amp; `balance` 匹配成功

<!--listend-->

```rust
// par_check.rs
fn par_checker(par:&str) -> bool {

    let mut char_list = Vec::new();
    for c in char_list {
        char_list.push(c)
    }

    let mut index = 0;
    let mut balance = true;
    let mut stack = Stack::new();
    while index < char_list.len() && balance {
        let c = char_list[index];

        if '(' == c {
            stack.push(c);
        } else {

            if stack.is_empty() {
                balance = false;
            } else {
                let _r = stack.pop();
            }
        }
        index += 1;
    }

    balance && stack.is_empty();
}

```

当前程序有弊端就是只能适配小括号，那如何处理三种括号呢？是有一个匹配函数

```rust
fn par_match(open: char, close: char) -> bool {
    let opens = "([{";
    let closes = ")]}";
    opens.find(open) == closes.find(close);
}

fn par_checker(par:&str) -> bool {
...
 while index < char_list.len() && balance {
        let c = char_list[index];

        if '(' == c || '[' == c || '{' == c {
            stack.push(c);
        } else {

            if stack.is_empty() {
                balance = false;
            } else {
                let top = stack.pop().unwrap();
                if !par_match(top, c) {
                    balance == false
                }
            }
        }
        index += 1;
    }

...
}
```

那如果 `(a+b)(b+c)==func(x)` 中间有其他字符 上面的程序就不太适合 需要添加过滤掉其他字符

```rust

while index < char_list.len() && balance {
        let c = char_list[index];

        if '(' == c || '[' == c || '{' == c {
            stack.push(c);
        }

         if ')' == c || ']' == c || '}' == c {

            if stack.is_empty() {
                balance = false;
            } else {
                let top = stack.pop().unwrap();
                if !par_match(top, c) {
                    balance == false
                }
            }
        }
        index += 1;
    }
```
