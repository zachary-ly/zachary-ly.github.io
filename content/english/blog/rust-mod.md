+++
title = "工厂方法模式"
date = 2025-03-19T09:47:00+08:00
tags = ["rust"]
categories = ["设计模式"]
draft = false
IMAGE = "/images/pl-banner-1.jpg"
AUTHOR = "LW"
+++

> `工厂方法模式` 是一种创建型设计模式，在父类中提供一个创建对象方法，允许子类
> 决定实例化对象的类型


## 动态分派 {#动态分派}

-   动态分派和运行时有关,也就是 `运行时多态` ，比如java中的 `方法重写` 具体创建类型由子类在运行时决定
-   延迟绑定
-   扩展性强： 秩序添加新的子类工厂
-   解偶：依赖抽象工厂和抽象产品，与具体实现解偶
-   运行时决策：通过多态机制，实际调用的工厂方法在运行时确定


## 静态分派 {#静态分派}

-   静态分派则是在编译时确定，比如 `方法重载`
-   提前绑定
    ```java
    class StaticFactory {
        public static Product Createor(String type) {
            if ("a".equals(type)){
                return new ProductA();
            } else {
                return new ProcutB();
            }
        }
    }
    ```

    -   高效： 类型决策在编译时完成，无需运行时动态开销

    -   紧耦合： 新增类型需要修改工程类的分支逻辑（违反开闭原则）

    -   编译时决策： 对象的创建逻辑由参数或者静态配置决定


## 区别和适用场景 {#区别和适用场景}

| 维度 | 动态分派    | 静态分派         |
|----|---------|--------------|
| ---  | ---         | ---              |
| 决策时机 | 运行时（动态多态） | 编译时（静态多态） |
| 扩展性 | 高（子类扩展） | 低（工厂类）     |
| 耦合度 | 低（依赖抽象） | 高（根据具体类型策略） |
| 典型模式 | 工厂方法模式 | 简单工厂模式     |
| 场景 | 灵活扩展，多态创建对象 | 类型固定，无需频繁扩展的简单场景 |
|      |             |                  |


## 设计取舍 {#设计取舍}

1.  选择动态分派：当系统需要支持未来扩展，且希望避免修改现有代码时。

2.  选择静态分派：当对象类型固定且无需动态扩展，或对性能有严格要求时。

> 动态分派和静态分派在工厂方法模式中代表了两种不同的设计哲学：动态分派通过多态实现运行时灵活性，是工厂方法模式的核心机制；静态分派通过编译时逻辑实现简单决策，通常用于轻量级场景。理解二者的区别有助于在设计中权衡扩展性、耦合度和性能。


## RUST 实现 {#rust-实现}

{{< figure src="/ox-hugo/factory-partten.png" >}}


### 代码实现 {#代码实现}

```rust
pub trait Button {
    fn render(&self);
    fn on_click(&self);
}

/// Dialog has a factory method
/// creates different buttons deppending on a factory implementation
pub trait Dialog {
    /// factory method to create button.
    fn create_button(&self) -> Box<dyn Button>;

    fn render(&self) {
        let button = self.create_button();
        button.render();
    }

    fn refresh(&self) {
        println!("Dialog: refresh");
    }
}

pub struct HtmlButton;

impl Button for HtmlButton {
    fn render(&self) {
        println!("HtmlButton: render");
        self.on_click();
    }

    fn on_click(&self) {
        println!("HtmlButton: on_click");
    }
}

pub struct HtmlDialog;
impl Dialog for HtmlDialog {
    fn create_button(&self) -> Box<dyn Button> {
        Box::new(HtmlButton)
    }
}

pub struct WindowsButton;
impl Button for WindowsButton {
    fn render(&self) {
        println!("WindowsButton: render");
        self.on_click();
    }

    fn on_click(&self) {
        println!("WindowsButton: on_click");
    }
}

pub struct WindowsDialog;
impl Dialog for WindowsDialog {
    fn create_button(&self) -> Box<dyn Button> {
        Box::new(WindowsButton)
    }
}

fn initialize() -> &'static dyn Dialog {
    let os = cfg!(windows);
    if os {
        println!("WindowsDialog initialized");
        return &WindowsDialog;
    } else {
        println!("HtmlDialog initialized");
        return &HtmlDialog;
    }
}

fn main() {
    let dialog = initialize();
    dialog.render();
    dialog.refresh();
}

```


### 静态分派 {#静态分派}

{{< figure src="/ox-hugo/static-factor.png" >}}

```rust
/// static factory method
///

pub trait Room {
    fn render(&self);
}

/// Maze game has a factory method producing different rooms
pub trait MazeGame {
    type RoomImpl: Room;
    /// factory method to create room.
    fn rooms(&self) -> Vec<Self::RoomImpl>;

    fn play(&self) {
        for room in self.rooms() {
            room.render();
        }
    }
}

/// the client code initializes resoureces an does other preparations
/// then it uses a factory to construst and run the game
pub fn run(maze_game: impl MazeGame) {
    println!("Game is running");
    maze_game.play();
}

//--------------------------------------------------------------------------
//--------------------------------------------------------------------------
#[derive(Debug, Clone)]
pub struct MagicRoom {
    title: String,
}

impl MagicRoom {
    pub fn new(title: String) -> Self {
        Self { title }
    }
}

impl Room for MagicRoom {
    fn render(&self) {
        println!("MagicRoom {}: render", self.title);
    }
}

pub struct MagicMaze {
    rooms: Vec<MagicRoom>,
}

impl MagicMaze {
    pub fn new() -> Self {
        Self {
            rooms: vec![
                MagicRoom::new("room1".to_string()),
                MagicRoom::new("room2".to_string()),
                MagicRoom::new("room3".to_string()),
            ],
        }
    }
}

impl MazeGame for MagicMaze {
    type RoomImpl = MagicRoom;

    fn rooms(&self) -> Vec<Self::RoomImpl> {
        self.rooms.clone()
    }
}

//--------------------------------------------------------------------------
//--------------------------------------------------------------------------

#[derive(Debug, Clone)]
pub struct OrdinaryRoom {
    id: i32,
}

impl OrdinaryRoom {
    pub fn new(id: i32) -> Self {
        Self { id }
    }
}

impl Room for OrdinaryRoom {
    fn render(&self) {
        println!("OrdinaryRoom {}: render", self.id);
    }
}

pub struct OrdinaryMaze {
    rooms: Vec<OrdinaryRoom>,
}

impl OrdinaryMaze {
    pub fn new() -> Self {
        Self {
            rooms: vec![
                OrdinaryRoom::new(1),
                OrdinaryRoom::new(2),
                OrdinaryRoom::new(3),
            ],
        }
    }
}

impl MazeGame for OrdinaryMaze {
    type RoomImpl = OrdinaryRoom;

    fn rooms(&self) -> Vec<Self::RoomImpl> {
        let mut rooms = self.rooms.clone();
        rooms.reverse();
        rooms
    }
}

//--------------------------------------------------------------------------
//--------------------------------------------------------------------------
fn main() {
    let ordinary_maze = OrdinaryMaze::new();
    run(ordinary_maze);

    let magic_maze = MagicMaze::new();
    run(magic_maze);
}


```
