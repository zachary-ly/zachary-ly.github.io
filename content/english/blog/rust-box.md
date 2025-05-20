+++
title = "Rust:智能指针Box<T>"
date = 2025-03-20T10:09:00+08:00
tags = ["rust"]
categories = ["rust"]
draft = false
IMAGE = "/images/pl-banner.jpg"
AUTHOR = "LW"
+++

`BOX<T>` 是一个用于在堆上分配内存的智能指针，提供对堆内存的所有权管理。


## 核心概念 {#核心概念}

1.  堆分配

    `Box<T>` 将数据存储在堆 `Heap` 而非栈 `Stack` 上，栈上仅存储指向堆数据的指针
2.  所有权机制

    `Box<T>` 拥有指向数据的所有权，当 `Box` 离开作用域时，会自动释放堆内存 通过 `Drop trait`
3.  固定大小的指针

    无论 `T` 的大小如何， `Box<T>` 本身的大小是固定（一个指针的大小） 编译器可以处理需要已知大小的场景


## 常见使用场景 {#常见使用场景}

1.  递归类型

    当类型包含自身，直接定义会导致无限大小，通过 `Box<T>` 固定指针大小解决
    ```rust
    struct ListNode {
        value: i32,
        next: Option<Box<ListNode>>,
    }
    ```
2.  大数据 避免栈溢出

    将大型数据，数组，集合类型存储在对上，避免栈内存不足
    ```rust
    let large_data = Box::new([0u8; 1_000_000]);

    ```
3.  Trait

    当需要返回多种实现了同一个 `trait` 的类型时， 使用 `Box<dyn Trait>` 实现动态分发
    ```rust
    trait Animal {
        fn speak(&self);
    }

    struct Dog;
    impl Animal for Dog { /* ... */ }

    struct Cat;
    impl Animal for Cat { /* ... */ }

    fn get_animal(name: &str) -> Box<dyn Animal> {
        match name {
            "dog" => Box::new(Dog),
            "cat" => Box::new(Cat),
            _ => panic!("Unknown animal"),
        }
    }

    ```
4.  转移所有权，无需复制数据

    使用 `Box<T>` 转移数据所有权，避免深拷贝
    ```rust
    let data = Box::new(2);
    let new_owner = data;

    ```
5.  Box&lt;T&gt; VS Rc&lt;T&gt;/Arc&lt;T&gt;

    `Box<T>` 是独占所有权，而 `Rc/Arc` 是引用技术共享所有权
