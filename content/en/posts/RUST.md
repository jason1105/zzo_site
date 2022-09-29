---
title: "Rust Programming Language"
date: 2021-06-19T23:52:00+08:00
tags: [study, note]
draft: false
description: ""
source: ""
hideToc: false
enableToc: true
enableTocContent: true
author: JasonLv
authorEmoji: 👾
tags:
- rust
categories:
- Book
- Programming Language
series:
- Rust Programming Book
image: images/feature1/markdown.png
---

# RUST

choco install visualstudio2019buildtools

注意 windows 下定位安装文件最好从具体盘符开始.


# 镜像资源

新建一个文件 ~/.cargo/config

```toml

[source.crates-io]
replace-with = 'tuna'
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

```


# 构建帮助文档

cargo doc --open

# 语法

## 语法
表示静态方法调用, 例如: `String::new()`, 这个静态方法是定义在 structure 中的一个 function, 常常用来生成一个 struct 的实例, 注意 struct 中的 method 和 method 的区别. `::` 既可以用于关联 function, 也可以用于 module 的 namespace.

## 语法

声明方法返回值.

## 语法

match 的匹配项的返回值.

# 03

## 03 01 [^8]



注意, 常量永远是 immutable. const 类型必须标明类型.

将所有硬编码变量声明为 const.

shadowing
使用 let 可以将一个 immutable 变量重新赋值. 

> *为什么使用 Immutable?*
> 
> 如果我们认为一个变量不会改变, 基于此写了代码A, 但在未来的某个时间点它被另一段代码B改变了, 就会导致代码A的行为超于预期, 由此产生的Bug是很难处理的. 所以使用了 immutable , 这样一来, 由 rust 保证这个变量是不变的, 您无需跟踪变量是否被改变就能保证代码A的正确性. 另外, immutable也用来改善并发程序.

## 03-02
无符号比有符号的正整数范围大一倍+1

数字默认是 i32

Number literals	Example
Decimal	98_222
Hex	0xff
Octal	0o77
Binary	0b1111_0000
Byte (u8 only)	 b'A'


通过模式匹配分解 tuple 

fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}

元组
组合不同类型的元素

数组
数组分配在 stack 上. 是不变的.
用来定义年份等.
let a: [i32; 5] = [1,2,3,4,5];
let a = [3; 5];

vector
可变的.

## 03 03
方法定义没有顺序
方法参数必须声明其类型.

let x = (let y = 6); 这样的语句是错误的. 因为一个声明块没有返回值.

let x = y = 3; 也是错误的.

表达式是用的最多的, 在 rust 中.

let x = 6; // 这里的 6 是一个表达式.

{} 这也是一个表达式
通常在方法的最后一行返回， 也可以使用 return 提前返回. 

## 03 05
if 必须要{}
if 的{} 可以是代码块, 也可以表达式.

loop 中的 break 可以将值返回.

# 04

## 04 01 ownership[^1]

复杂的数据类型保存在堆上, 有自己的 owner.

下面这些数据类型直接保存在栈上, 没有 owner:

![[Pasted image 20210704183028.png]]


调用方法b, 方法b会将本地变量保存在栈上.

ownership作用
- 跟踪哪些程序用了哪些变量在堆上
- 减少堆上的重复数据
- 清理堆上无用的数据

ownership规则, 3个
1. 变量都有owner
2. 同一时刻只能有一个
3. 当owner结束时, 变量也被丢弃

```rust
let s = "hello";
let s = String::from("hello");
```

前者是字面量, 不可变, 即使 mut也不行, 因为它分配在栈上, 它有固定大小.
后者存在堆上, mut 后可变.

一个变量超出 scope 后(离开方法{}之后), rust 调用 drop 函数, 变量的持有者在 drop 函数中释放变量所占有的内存.

复合变量的复制实际上移动了 owner. move 可以从 immutable 变成 mutable.

```
let s1 = String::from("hello");
let s2 = s1;
// s1 已经无效了.
// 方法调用也是同样的, 例如
let s3 = String:from("hello");
foo(s3);
fn foo(s: String) { ... }
//  println!("s3: {}", s3); // error
```

上面的问题使用 clone() 可以解决, clone 会在堆上重新分配一个与 s1 具有相同内容的对象. (但不是同一个对象).

```rust
let mut a = String::from("hi");
let b = a.clone();
a = String::from("hello");
println!("a {}", a); // a hello
println!("b {}", b); // b hi
```


调用完 foo 后, s3 变量也无效了. 如果还想使用 s3, 可以在foo() 方法中返回一个元组, include the parameter of input.

```rust
fn foo(s: String) {
  let size = s.len(); // 模拟一个业务
  (s, size) // 返回业务结果, 同时返回 s
}

let s4 = "a";
s4 = foo(s4).0;
```

这里产生了一个问题, 每次调用方法都要返回原始参数值.

How we resolve this problem.


> *栈和堆*
>
> *分配*
> 
> 栈上的数据必须有确定的大小.
> 
> 对于那些未知大小的数据或者可能在运行时改变大小的数据, 则必须保存在堆上.
> 
> 栈上分配更快, 因为无需查找可用空间.
> 
> 在堆上分配则需要查找到足够的空间, 还要记录每个数据所占用的空间以便进行下一次分配
> 
> *使用*
> 
> 栈的读取更快.
> 
> 堆的读取则需要跟随指针进行跳转. (**指针可以分配在栈上, 因为指针的大小是固定的.**)
> 
> 当调用一个函数的时候, 传递给函数的指针(对变量的引用)以及函数本身的局部变量都会被压入栈, 在函数体结束的时候, 它们都会被弹出栈.
> 
> Ownership 要处理的问题就是: 跟踪那些使用了堆空间的代码, 减少堆中重复的数据, 清理堆中无用的数据, 从而避免内存溢出.
> 
> 理解 Ownership 存在的意义就是为了管理堆, 可以帮助你理解代码工作的方式.



## 04 02 reference

Think about reference in Java. Just post reference of object when call a method.

String a = new String 

Ref in Rust is a reference pointed the owner of original variable.

```
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}

```

![[Pasted image 20210721232715.png]]

s -> s1 -> data in memory

s is not owner of the object on heap. s1 is owner.

remember that s is immutable, so you can't change the value in calculate_length() unless use &mut.

**Mutable reference**

to address data races, Rust don't allow use &mut more than one time with same variable.

scenario:
- multiple read was allowed
  let mut s = ...
  let r = &s;
  let r1 = &s;
- mix read and write wasn't allow
  let mut s = ..

  let r1 = &s; // no problem
  let r2 = &s; // no problem
  let r3 = &mut s; // BIG PROBLEM

上述方式解决了同步变量的问题. 可以同时读, 但不能同时写, 即在一个 scope 中, 只能有一个写操作, 离开作用域后, 变量无效, 例如:

对于 i32 等原始的类型, 直接赋值就是 copy, 保存在堆栈上. 使用 & 则是引用, Rust 把 & 表述为 borrow, 临时借用一下, 所以原始值你不能变哦, 看下面的例子

```rust
    let mut x = 5;
    let y = x; // copy value of x to y, store on stack
    println!("y = {}", y);
    x = 6;
    println!("y = {}", y);

    
    let mut x = 5;
    let y = &x; // reference to x, y is borrow of x
    println!("y = {}", y);
    x = 6;  // x is borrowed, and will use in the future, so it won't be modify
    println!("y = {}", y); // here, use y that is borrow of x
```

编译出错, 因为 x 已经 borrow 出去了, 所以 `x = 6` 是非法的操作.

Borrow rules: ^79ef45
- There are only one mutable reference or any number of immutable exists at one time. (but not both of)
- Reference must be valid.

```rust
/*
let mut y = String::from("1");
y 的数据类型(指向的内存区域的类型)是 String, 
y 是资源 String::from("1") 的所有者(owner), 
在编译期, y 被替换为真正保存了 "1" 这个数据的内存地址. 例如: 0x09e3a317
因为使用了 mut, 所以 y 区域的内容可以修改, 
换句话说, 因为使用了 mut, 所以 "1" 所在的内存区域是可以被编辑的.
总结: 申请了一块可以编辑的内存标记为 y, 里面存放了 String::from("1")
*/
let mut y = String::from("1");  // 注意, y 是 owner, 决定了什么时候可以释放内存. 即 0x09e3a317 地址一旦离开作用域, 将被回收.
y = String::from("11");         // 编辑内存中的内容
println!("y: {}", y);   // y: 11
println!();

/*
& 在 Rust 中叫 borrow, 暂时借用, 并不是变量的主人, 所以不拥有变量. 
& 可以理解为 c语言 中的取地址操作, 例如:
&y 的意思是: 借用一下 y. 那怎么借用呢? 我只要 y 的地址, 通过 y 和数据打交道, 所以 y 还是数据的owner, 我只是借用一下,
&mut y 的意思是, 借用一下, 同时还有修改数据的权限
例如:
let a = &y; 的意思是, 申请了一块只读内存, 标记为a, 里面存放的是 y 的地址(且不能修改成别的地址). a 只可以使用但不能修改 y (这个 a 不就是指针嘛)
let a = &mut y; 的意思是, 申请了一块只读内存, 标记为a, 里面存放的是 y 的地址(且不能修改成别的地址). a 有权修改 y 的内容( a 是一个可变指针) 
let mut a = &y; 的意思是, 申请了一块可读写内存, 标记为a, 里面存放的是 y 的地址(也可以修改成存放 z 的地址). a 只可以使用但不能修改 y (这个 a 不就是指针嘛)
let mut a = &mut y; 的意思是, 申请了一块可读写内存, 标记为a, 里面存放的是 y 的地址(也可以修改成存放 z 的地址), a 有权修改 y 的内容( a 是一个可变指针) 

borrow 有两个规则:
1. 同一时刻, 一个资源只能有一个 &mut borrow 或者多个 & borrow, 
2. 引用必须是有效的. 即不能引用一个不存在的东西.

borrow 的两个规则可以可以理解为读写锁, 
	在一个作用域(事务)当中, 对资源进行锁定以避免竞争条件, 要么都是读锁(s), 要么只有一个写锁(x).
	& 理解为读锁.
	&mut 理解为写锁.
这样设计的好处: 避免一段代码中不同的逻辑之间的干扰(类似事务之间的干扰)
隐含的 borrow: 在方法参数中, 例如 println!()
*/
{
	println!("-------------- Block A");
	let a = &mut y;             // a 的数据类型(指向的内存区域的类型)是 &mut String, 通过 mut 引用 y (拥有了 y 的 x 锁), 
	// let mut c = &mut y;      // c 也想要 y 的 x 锁, 这是不可能的, y 上只能加一把 x 锁. 已经被 a 占用了, 且 a 还没有释放.
	// y = String::from("111"); // owner 当然有最大权限, 但此时因为 y 上有一把 x 锁被 a 占用, 且 a 还没有释放, 所以 owner 也无权修改.
	// println!("y: {}", y);    // 这里的 y 作为了方法参数, 是 & borrow (相当于 s 锁), 因为 y 上有 x 锁, 这个 borrow 是不被允许的.
	*a = String::from("2");     // a 拥有 x 锁, 可以写 y. 此后 a 作用域结束, 释放了 x 锁.
	println!("y: {}", y);       // 现在 y 上没有任何锁, y: 2
	y = String::from("111");    // 因为 a 的作用域已经结束, y 上没有锁, 所以现在 owner 可以修改了.
	let mut c = &mut y;         // y 上没有锁, c 现在可以加一把 x 锁
	//println!("y: {}", y);     // y 上有 c 加的锁, 且 c 还没有释放, 所以不能再 borrow
	*c = String::from("3");     // c 修改了 y, 此后 c 作用域结束, 释放 x 锁.
	println!("y: {}", y);       // 现在 y 上没有任何锁, y: 3
	println!();
} 

{
	println!("-------------- Block B");
	let a = & y;                // a 拥有了 y 的 s 锁
	// let c = &mut y;          // c 想要 y 的 x 锁, 这是不可能的, 因为 s 和 x 不能共存, a 已经持有 s 锁且 a 还没有释放, 所以无法加 x 锁.
	// y = String::from("111"); // owner 当然有最大权限, 但此时因为 y 上有一把 s 锁, 且还未释放, 所以 owner 也无权修改.
	let c = & y;                // 但 c 可以得到 y 的 s 锁, 共享锁可以加多次, 不多 c 并没有使用, 所以随即这把 s 锁就失效了
	println!("y: {}", y);       // 这里的 borrow 相当于 s 锁, 因为 s 锁可以加多次, 所以没有问题. y: 3
	println!("a: {}", a);       // 此后 a 作用域结束, 释放了 s 锁. a: 3
	y = String::from("111");    // y 上没有任何锁, y 是owner, 当然可以修改
	println!("y: {}", y);       // y: 111

	println!();
}
```

## 04 03 slice

A __string slice__ is a reference to part of a `String`

slice data structure? #todo 

`&str` is string slice type.

Slice is a pointer pointing to a specific point.

```
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];  // Remeber to use &
    let world = &s[6..11];
}

```

![[Pasted image 20210704163935.png]]

# 05 Structs

`self` in implementation of struct means the object itself or current module.

`Self` is a type of module or struct. `&self`  is a shorthand of `self : &Self`

> `self` when used as first method argument, is a shorthand for `self: Self`. There are also `&self`, which is equivalent to `self: &Self`, and `&mut self`, which is equivalent to `self: &mut Self`.
> 
> `Self` in method arguments is _syntactic sugar_ for the receiving type of the method (i.e. the type whose `impl` this method is in). This also allows for generic types without too much repetition.
> 
> https://stackoverflow.com/questions/32304595/whats-the-difference-between-self-and-self

## 声明

特殊情况

```
fn main() {
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}

```

# 06 enumeration

## 什么是枚举

枚举是一种数据类型(可以想象成一个类), 这个类包含一个有边界的该种类型的数据集, 该数据集不能动态扩展.

例如: 下面的 Person 包含了一个 Person 类型的数据集, 其中有两个元素 MAIL 和 FEMAIL.

```
enum Person {
	MAIL,
	FEMAIL,
}
```

可以给类型中添加元素:

```
enum Person {
	MAIL("Lee"),
	FEMAIL("Lily"); // NOTICE here, use ; instead of , 
	
	String name;
	private Person(String name) {
		this.name = name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return this.name;
	}
}
```

> 枚举的使用: 当定义的数据类型存在不同的子类型时, 例如: 定义一个返回值 Result 类型, 且这个返回值类型有正常和异常两种情况, 这种场合就是一个典型的枚举.
	> ```rust
	> enum Result<T, E> {
	>     Ok: T,
	>     Err: E,
	> }
	> ```

# 07 Package, Crates, and Modules

-   **Packages:** A Cargo feature that lets you build, test, and share crates. cargo 建立的一个工程是一个 package. 一个 package 包含一个或者多个 crate. 一个 package 只能包含 0 个或者 1 个 Lib crate, 但可以包含多个二进制可执行文件 crate. 且一个 package 必须有一个 crate.
-   **Crates:** A tree of modules that produces a library or executable. 一个 crate 是一个可执行文件或者一个库文件, **crate 相当于一个命名空间**. 里面包含 module 的树形结构. main.rs 是一个 crate 的根. lib.rs 也是一个 crate 的根. Rust 默认 main.rs 和 lib.rs 都是 crate, 且名字和 package in Cargo.toml 一样. 一个 package 可以多个二进制 crates, 只要把它们放在 `src/bin` 下即可.
-   **Modules** and **use:** Let you control the organization, scope, and privacy of paths. 提供一个逻辑的分层. **注意: 所有模块都指向根模块 `crate`**
-   **Paths:** A way of naming an item, such as a struct, function, or module

## 07 01

- Package: cargo.toml 文件所在项目的名字, 定义在 cargo.toml 的 `[package].name` 中.
- Binary crate: 可执行 crate
- Library crate: 库 crate

- 新建一个 binary: `cargo new <package_name>`
- 新建一个 library: `cargo new --lib <package_name>`


包的绝对引用和相对引用:
- 绝对引用: 如果 lib 是固定的, client 有可能变化, 用绝对引用
- 相对引用: 如果 lib 和 client 的相互关系是稳定的, 用相对引用

https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#:~:text=The%20reason%20is%20that%20child%20modules%20wrap%20and%20hide%20their%20implementation%20details%2C%20but%20the%20child%20modules%20can%20see%20the%20context%20in%20which%20they%E2%80%99re%20defined

## 07 04 use

use 可以用于

- module
- struct
- enum
- function : 注意很少使用 use 来导入一个 function. 因为使用的时候容易产生混淆.

```
use std::cmp::Ordering;
use std::io;
```
equals ->
```
use std::{cmp::Ordering, io};
```


```
use std::io;
use std::io::Write;
```
equals ->
```
use std::io::{self, Write};  // use std::io and std::io::Write
```

```
use std::collections::*;  // used for test
```

导入一个模块的场合, 直接在根目录下加一个文件 front_of_house.rs, 然后在 lib.rs 中使用 `mod` :

mod front_of_house;

如果 front_of_house 中还包含单独文件的子模块, 则那个子模块必须放在 front_of_house 文件夹中. 


```
/-
 |   lib.rs
 |   front_of_house.rs
 |---front_of_house
	   front_of_house.rs
```

这样的原因是因为 rust 的 module 是 full qulification. 也就是说存在 这样的 module:
- io.rand
- app.rand

如果把每个模块单独定义在文件中, 就会出现两个名为 rand.rs 的文件.

`crate` 可以作为相对路径中的根节点. 注意: 一个 package 中的多个 crate 具有相同的名字, 所以 `crate` 指的是当前文件所在 crate 的根.

# 08

## 08 01 Vector

Vector 中的元素使用的时候加上 `*`.
```rust
fn main() {
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}
```

## 08 02 String

rust 核心只有一种 string type, 那就是 str. 而 String 类型是标准库函数提供的. 标准库也提供其他类型的 string type, 例如: OsString OsStr CString CStr

## 08 03 HashMap

对于实现了 Copy 的类型, 例如 i32, HashMap 持有副本, 对于 Ownerable 的类型, 例如 String, 会将 owner 交给 HashMap.

更新已有值, 下面的例子是统计单词出现的次数:

```rust
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0); // return the value if exist, or insert new value and return 0
        *count += 1;
    }

    println!("{:?}", map);
```


entry()  和 or_insert() 返回的是 &mut v , 通过 * 解引用后这个值可以直接修改.


# 09

如果正在写一个函数, 当错误发生时, 不要 panic, 而是把错误返回.


# 10 Generic Types, Traits, and Lifetimes[^2]

trait 两个作用:

- Trait: 定义行为接口
- Trait Bound: 约束泛型, 例如 <T: Display>, 指的是实现了 Display trait 的类型

Rust 使用了单态化, 所有的泛型在编译期会被替换成实际的类型. 也就是说, 如果一个泛型方法被调用, Rust 会维持多个实例方法对应着每个调用. 这样就不会因为使用了泛型而牺牲性能. (但是我想, 这样牺牲的是空间, 因为泛型参数的增加可能导致在编译期生成很多实例.)

## traits[^3]

约束条件: trait 和 待绑定的结构必须至少有一项是 local的. 也就是说, 不能将第三方库中的 trait 与第三方库中的结构绑定在一起.


trait 中的默认方法可以调用 trait 当中的其他方法, 即使那些方法抽象的. 这和抽象类有些类似. 如果方法 a() 是一个默认方法, 我们覆盖了 a(), 则不能在我们的方法当中调用默认方法 a().

Trait Bound

限制泛型的范围. 可以理解为: 要求一个参数(任意类型)具有指定的特性.

```rust
// single
pub fn notify(item1: &impl Summary, item2: &impl Summary) { // both item1 and item2 must have implemented Summary

pub fn notify<T: Summary>(item1: &T, item2: &T) { // Same as previous notify()

// multiple
pub fn notify(item: &(impl Summary + Display)) { // item must has implemented Summary and Display

pub fn notify<T: Summary + Display>(item: &T) { // Same as previous notify()

// use where clause
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 { // t must has implemented trait of Display and Clone, u must has implemented trait of Clone + Debug.

fn some_function<T, U>(t: &T, u: &U) -> i32
   where T: Display + Clone, U: Clone + Debug { // convenient implements
```

return a trait

```
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn returns_summarizable() -> impl Summary {  // HERE
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}

```

## 10 03 The Borrow Checker[^4]

泛型生命周期语法

`&'<lifetime> [mut] <type>`

例如:

```
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

生命周期本身没有意义, 其意义在于指出各个参数之间的生命周期的关系. 返回值生命周期是 input lifetime 的交集.

Rust 根据三个规则推断 lifetime in function or method declaration

1. 每一个引用的参数默认有自己的生命周期. 例如: 对于 `fn foo(x:&i32, y: &i32)` 来说, Rust 默认会为每个参数指定一个生命周期: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`.
2. 如果只有一个输入参数, 则所有的输出参数与输入参数的生命周期保持一致. 例如: 对于 `fn foo(x: & i32) -> &i32` 来说, Rust 自动指明生命周期 `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. 如果输入参数中有 `&self` 或者 `&mut self`, 则输出参数的生命周期与保持一致. 这样一来, 对于所有包含了 `&self` 或者 `&mut self` 的 method 来说, 生命周期的声明就可以省略了. 

`'static` means a lifetime duration same as whole application. All string literal have this lifetime.

struct 的生命周期

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

ImportantExcerpt 和其字段 part 的生命周期一致.

# 11 test[^5]

## 11 01 Write a test

自定义的 struct 和 enums 需要实现 PartialEq 和 Debug, 前者用来比较, 后者用来输出. 可以使用 `#[derive(PartialEq, Debug)]` 在自定义的 struct 和 enums 上.

- `assert!()`
- `assert_eq!()`
- `assert_ne!()`
- `#[should_panic(expected = "")]`


```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"),
                "Greeting did not contain name, value was `{}`",  // format!
                result);
    }
```

## 11 02 Controll a test[^6]

指定测试线程数量

指定线程数量可以避免并行测试对共享资源的读写造成的失败.

```
$ cargo test -- --test-threads=1
```


输出

```
$ cargo test -- --show-output
```


执行部分测试

```
$ cargo test <str>
```

只要包含了 `str` 的测试方法都会被执行. 例如下面执行了包含 `bu` 的测试方法, 测试报告中同时显示 `8 filtered out`, 说明有 8 个测试方法没有执行.

```shell
$ cargo test gu

running 2 tests
test tests2::guess_panic ... ok
test tests::guess_panic ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 8 filtered out; finished in 0.00s

```


忽略测试

```rust
#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

We can specify `cargo test -- --ignored` to include ignored test.

## 11 03 Integration test

- Make dir `tests` in root of project.
- Add rs file in `tests`
- Add annotation `[test]` on every test method.
- Notice that every rs file is an independent crate.
- Run particular test using `cargo test --test <name>`

```rust
// Listing 11-13: An integration test of a function in the adder crate
use controlling_how_test;

#[test]
fn it_adds_two() {
    assert_eq!(4, controlling_how_test::add_two(2));
}
```

**Use setup of common**

- Create a common dir in tests
- Create mod.rs in common
- Import module of common by using `mod common;`

```rust
// Listing 11-13: An integration test of a function in the adder crate
use controlling_how_test;
mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, controlling_how_test::add_two(2));
}
```


# 12 Command Line Program

see program.

# 13 Closure and Iterator

## 13 01 Closure

函数不能捕获此法作用域, 闭包可以捕获此法作用域.

```rust

fn main() {
	let x = 4;
	fn hello() {
		println!("x: {}", 4); // error
	}

}


```

一个闭包只能推断绑定到一种类型, 如果是两种, 编译出错, 例如:

```rust
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
```

## 13 02 Iterators

调用 ite.next() 需要 mutable, 因为其内部状态会变化, 使用 for 则不会

vec.iter() 返回 immutable,  vec.into_iter() 返回 mutable.

# 14 Cargo and Crates.io

## 14 02 Publishing

文档注释 `///`, 文档注释中的测试代码可以通过 `cargo test` 运行.

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

```bash
$ cargo test

   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

模块注释 `//!`

## 14 02  Cargo

Run particular package in a workspace.

```bash
$ cargo run -p <package>
```

Test particular package in a workspace.

```bash
$ cargo test -p <package>
```

## 14 04 Installling binaries

```shell
cargo install <binary name>
```

# 15 Smart pointers

普通的 reference 只是 borrow 了 data, 而 smart pointer 则可以是 owner.

Smart pointer implements `Deref` and `Drop` trait.

## 15 01 Box[^7]

Box 将数据保存在 heap, 同时在 stack 上保存一个指针, 指向 heap 中的数据.

对于递归的结构

let list = Cons(1, Cons(2, Cons(3, Cons(4, nil))))

![[Pasted image 20210719122643.png]]

Rust 无法知道 list 的实际大小.

改用 Box

let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Cons(4, Box::new(nil))))))))

![[Pasted image 20210719122659.png]]

因为 Box 是一个指针, 有固定的大小, list 其结构不在是递归嵌套, 而是一个链表.

Box 的使用场景:
-   When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
-   When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
-   When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type

**Check**

## 15 02 Deref Trait

使用  * (dereference operator) 可以沿着指针得到具体的值. 对于自定义的类型, 如果想直接使用 * , 可以实现 Deref 这个 trait.

```
struct Mybox<T>(T);

impl<T> Deref for MyBox<T> {
 	type Target = T;
	fn deref(&self) -> &Self::Target {
		...
	}
}

let y = MyBox::new(5);

```

当我们使用 `*y` 时,  Rust 实际上会这样调用 `*(y.deref())`

 Deref coercion 是自动发生的, 当传入的参数和方法的参数不一致时, Rust 会调用 deref 方法.

 Deref coercion 的三种情况(针对不同的  trait):
 
-   From `&T` to `&U` when `T: Deref<Target=U>`
-   From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
-   From `&mut T` to `&U` when `T: Deref<Target=U>`

## 15 03 Drop Trait

Must implement drop() in Drop trait.

Drop trait is used for deallocating memory when variables leave their scope.

All smart pointers had implemented Drop trait. 

## 15 04 Rc\<T>

允许多个 owner of immutable 存在.

如果一个值可能被多处引用. 且这些引用的存活时间不一定, 可以使用 `Rc<T>`. 

如果一个变量可能被多处引用, 在声明的时候使用 Rc, 使用的地方调用 `Rc::clone(&...)` 就行, Rc 的 dereference 是自动的. 所以基本上不需要考虑如何使用 Rc. 另外还有一个查看计数的方法 `Rc::strong_count(&...)`

对于自定义的结构体, 且有 impl 方法, 如果想用 RefCell 结合 Rc, 这样定义比较好:

```
enum List {
	Cons(i32, RefCell<Rc<List>>),
	Nil,
}

impl List {
	fn tail(&self) -> Option<&RefCell<Rc<List>>> {
		if let Cons(_, list) = self {
			Some(list)
		} else {
			None
		}
	}
}
```

```
enum List {
	Cons(i32, Rc<RefCell<List>>),
	Nil,
}

impl List {
	fn tail(&self) -> Option<&Rc<RefCell<List>>> {
		if let Cons(_, list) = self {
			Some(list)
		} else {
			None
		}
	}
}

if let Some(list) = a.tail() { // tail 方法找不到  method not found in `Rc<RefCell<tests::test99_25::List>>`
	...
}
```


但是 Rc 用于 Immutable 场合, Mutable 场合怎么办呢?

> `Rc<T>` 只能用于单线程.

## 15 05 RefCell\<T>

Rust 是保守的, 即使你准守了 Borrow 规则, 但 Rust 却无法确定代码的正确性, 因此会拒绝这些代码.

换句话说, `RefCell<T>` 可以将 borrow 规则的检查延迟到运行时, 这样一来, 你可以实现内部修改这一算法, 即修改那些 immutable 的变量, 但还是要遵守 borrow 规则. [[RUST#^79ef45]]

> `RefCell<T>` 只能用于单线程.

RefCell 有 Rc 的计数功能. 与 Rc 的计数不同, RefCell 通过跟踪对象上的 mut 和 immut 的数量来判断是否满足 borrow 规则 (borrow rules: 同一时刻只允许一个 mut 或者 多个 immut, 不允许 mut 与 immut 同时存在), 计数到0并不意味着释放内存. 

Here is a recap of the reasons to choose `Box<T>`, `Rc<T>`, or `RefCell<T>`:

-   `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>` have single owners.
-   `Box<T>` allows immutable or mutable borrows checked at compile time; `Rc<T>` allows only immutable borrows checked at compile time; `RefCell<T>` allows immutable or mutable borrows checked at runtime.
-   Because `RefCell<T>` allows mutable borrows checked at runtime, you can mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is immutable.


## 15 06 Reference Cycles - Weak\<T>


```

Rc::downgrade(&...)		// get Weak
Rc::weak_count(&...)	// get count
&obj.upgrade() 			// see if have value of Some<T>

```


# 16 Concurrency

## 16 01 Using Threads

```rust
thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
```

## 16 02 Tansfer Data Between Thread

```
let (tx, rx) = mpsc::channel(); // 

thread::spawn(|| {
	tx.send("hi").unwrap();
});

println!("Got: {}", rx.recv().unwrap());

```


## 16 03 Share State

Mutex is a mutual exclusion.

Use Arc instead of Rc in multiple threads.

```rust
    use std::thread;
	use std::sync::{Arc, Mutex};
	
    #[test]
    fn test16_13() {
        let counter = Arc::new(Mutex::new(0)); // Arc is atomically reference counted
        let mut handles = vec![];

        for _ in 0..10 {
            let counter = Arc::clone(&counter);
            let t = thread::spawn(move || {
                let mut cc = counter.lock().unwrap();
                *cc += 1;
            });

            handles.push(t);
        }

        for t in handles {
            t.join().unwrap();
        }

        println!("counter = {}", *counter.lock().unwrap());
    }
```

## 16 04 Send Trait And Sync Trait

All ownership of values of type implementing `Send` trait can safty tranfer between threads.

All values of type implementing `Sync` maker trait can be safty referenced from multiple threads.

All primitive types are `Send` and `Sync`, and any type composed entirely of Send are also Send, Sync is the same way.

通常不需要自己实现 Send 和 Sync, 







[^1]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
[^2]: https://doc.rust-lang.org/book/ch10-00-generics.html
[^3]: https://doc.rust-lang.org/book/ch10-02-traits.html
[^4]: [The Borrow Checker](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-borrow-checker)
[^5]: https://doc.rust-lang.org/book/ch11-01-writing-tests.html
[^6]: https://doc.rust-lang.org/book/ch11-02-running-tests.html
[^7]: https://doc.rust-lang.org/book/ch15-01-box.html
[^8]: https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html