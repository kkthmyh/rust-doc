[toc]

# Rust进阶

## 1 泛型

### 1.1 结构体中泛型
```rust
#[derive(Debug)]
//单个泛型
struct Point<T> {
    x: T,
    y: T,
}
#[derive(Debug)]
//多个泛型
struct Point2<T,U>{
    x:T,
    y:U,
}

fn main() {
    let point1 = Point {
        x: 10,
        y: 12,
    };
    println!("Point1 is {:#?}", point1);

    let point2 = Point2 {
        x: 10,
        y: 12.1,
    };
    println!("Point2 is {:#?}", point2)
}
```
### 1.2 枚举中使用泛型
```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
### 1.3 方法中使用泛型
#### 1.3.1 例1
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn get_x(&self) -> &T {
        &self.x
    }
    fn get_y(&self) -> &T {
        &self.y
    }
}

fn main() {
    let point = Point {
        x: 1.1,
        y: 2.2,
    };
    println!("x is {}", point.get_x());
    println!("y is {}", point.get_y());
}
```
#### 1.3.2 例2
```rust
struct Point2<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point2<T, U> {
    fn create_point<V, W>(self, other: Point2<V, W>) -> Point2<T, W> {
        Point2 {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point2 { x: "a", y: 'b' };
    let p2 = Point2 { x: 1, y: 1.1 };
    let p3 = p1.create_point(p2);
    println!("p3.x is {:#?},p3.y is {:#?}", p3.x, p3.y)
}
```
## 2 Trait
### 2.1 定义trait
```rust
// 定义trait
pub trait GetInformation {
    fn get_name(&self) -> &str;
    fn get_age(&self) -> u32;
}

// 定义结构体
struct Student {
    name: String,
    age: u32,
}

struct Teacher {
    name: String,
    age: u32,
    subject: String,
}

// 实现trait
impl GetInformation for Student {
    fn get_name(&self) -> &str {
        &self.name
    }

    fn get_age(&self) -> u32 {
        self.age
    }
}

impl GetInformation for Teacher {
    fn get_name(&self) -> &str {
        &self.name
    }

    fn get_age(&self) -> u32 {
        self.age
    }
}

// trait 作为参数
pub fn print_info(item: impl GetInformation) {
    println!("name is : {}", item.get_name());
    println!("age is : {}", item.get_age())
}

fn main() {
    let student = Student {
        name: String::from("xiaoming"),
        age: 18,
    };
    let teacher = Teacher {
        name: String::from("xiaohuang"),
        age: 40,
        subject: String::from("数学"),
    };

    println!("student name is {},age is {}", student.name, student.age);
    println!("teacher name is {},age is {}", teacher.name, teacher.age);

    print_info(student);
    print_info(teacher);
}
```

### 2.2 trait bound 
#### 2.2.1 改写上例中的print_info方法
```rust
// trait bound写法
pub fn print_info<T: GetInformation>(item: T) {
    println!("name is : {}", item.get_name());
    println!("age is : {}", item.get_age());
}
```
#### 2.2.2 多个trait bound的场景
```rust
struct Student {
    name: String,
    age: u32,
}

pub trait GetName {
    fn get_name(&self) -> &str;
}

pub trait GetAge {
    fn get_age(&self) -> u32;
}

impl GetName for Student {
    fn get_name(&self) -> &str {
        &self.name
    }
}

impl GetAge for Student {
    fn get_age(&self) -> u32 {
        self.age
    }
}

// 多个trait bound的情况
// 写法一
// pub fn print_info<T: GetName + GetAge>(item: T) {
//     println!("name is : {}", item.get_name());
//     println!("age is : {}", item.get_age());
// }

// 写法二
pub fn print_info<T>(item: T)
    where T: GetName + GetAge {
    println!("name is : {}", item.get_name());
    println!("age is : {}", item.get_age());
}


fn main() {
    let student = Student {
        name: String::from("xiaoming"),
        age: 18,
    };
    print_info(student);
}
```
#### 2.2.3 trait作为返回值的场景
```rust
pub fn produce_item() -> impl GetAge {
    Student {
        name: String::from("xiaoli"),
        age: 18,
    }
}
```

#### 2.2.4 有条件的实现方法

```rust
trait GetName {
    fn get_name(&self) -> &str;
}

trait GetAge {
    fn get_age(&self) -> u32;
}

struct PeopleMatchInformation<T, U> {
    master: T,
    student: U,
}

impl<T: GetName + GetAge, U: GetName + GetAge> PeopleMatchInformation<T, U> {
    fn print_all_info(&self) {
        println!("print all information");
        println!("master name = {}", self.master.get_name());
        println!("master age = {}", self.master.get_age());
        println!("student name = {}", self.student.get_name());
        println!("student age = {}", self.student.get_age());
    }
}

struct Teacher {
    name: String,
    age: u32,
}

impl GetName for Teacher {
    fn get_name(&self) -> &str {
        &self.name
    }
}

impl GetAge for Teacher {
    fn get_age(&self) -> u32 {
        self.age
    }
}

struct Student {
    name: String,
    age: u32,
}

impl GetName for Student {
    fn get_name(&self) -> &str {
        &self.name
    }
}

impl GetAge for Student {
    fn get_age(&self) -> u32 {
        self.age
    }
}

fn main() {
    let s = Student {
        name: String::from("xiaoming"),
        age: 17,
    };
    let t = Teacher {
        name: String::from("xiaoli"),
        age: 47,
    };

    let m = PeopleMatchInformation {
        master: t,
        student: s,
    };
    m.print_all_info()
}
```

####  2.2.5 有条件的实现trait

```rust
// 对任何实现了特定trait的类型有条件的实现trait
trait GetName {
    fn get_name(&self) -> &str;
}

trait PrintName{
    fn print_name(&self);
}

impl <T:GetName> PrintName for T{
    fn print_name(&self) {
        println!("name = {}",&self.get_name());
    }
}

struct Student {
    name: String,
    age: u32,
}

impl GetName for Student {
    fn get_name(&self) -> &str {
        &self.name
    }
}

fn main() {
    let s = Student {
        name: String::from("xiaoming"),
        age: 17,
    };
    s.print_name();
}
```

##  3  生命周期

###  3.1  相关概念

- Rust中的每个引用都有其生命周期，也就是引用保持有效的作用域。大部分时候生命周期是隐含并可以推断的。
- 生命周期的主要作用是避免悬垂引用。
- Rust编译器是通过借用检查器来检查生命周期是否有效。

```rust
fn main() {
    let a;
    {
        let b = 10;
        a = &b;
    }
    println!("a is {}",a);
}
// 报错信息
// 可以看到b在第6行后已经走出作用域，被drop了，但是a持有b的引用，显然是非法的，rust为了避免悬垂引用，此段程序无法正确运行
error[E0597]: `b` does not live long enough
 --> src\main.rs:5:13
  |
5 |         a = &b;
  |             ^^ borrowed value does not live long enough
6 |     }
  |     - `b` dropped here while still borrowed
7 |     println!("a is {}",a);
  |                        - borrow later used here
// 正确写法
fn main() {
    let a;
    let b = 10;
    a = &b;
    println!("a is {}", a);
}

```

###  3.2 方法中的生命周期

```rust
// 表明返回的传入参数的生命周期必须大于或者等于返回值的生命周期
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
// 对于确定的返回值情况
fn get_str<'a>(x: &'a str, y: &str) -> &'a str {
    x
}

fn main() {
    let s1 = String::from("java");
    let s2 = String::from("php");
    let l = longest(&s1, &s2);
    let str = get_str(&s1, &s2);
    println!("longest is {}", l);
    println!("str is {}", str);
}
```

###  3.3 结构体中的生命周期

```rust
#[derive(Debug)]
struct A<'a> {
    name: &'a str,
}

fn main() {
    let a = A{name:"hello"};
    println!("A is {:#?}",a);
}
```

###  3.4 生命周期省略

Rust编译器会根据三条规则检查何时不需要显式标明生命周期注解，当编译器检查完这三条规则后仍然不能计算出引用的生命周期，则会报错。

- 每个引用的参数都有自己的生命周期参数。例如：

  一个引用参数的函数，有一个生命周期：```fn foo <’a>(x:& ’a i32)```；

  两个引用参数的函数，有两生命周期：```fn foo <’a，’b>(x:& ’a i32，y:& ’b i32)```；

- 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数

  ```fn foo (x:& i32) ->i32``` 等价于```fn foo <’a>(x:& ’a i32) -> & ’a i32```

-  如果方法有多个输入生命周期参数，其中之一为```&self```或者```& mut self ```,那么self的生命周期参数被赋予所有输出生命周期参数

###  3.5 静态生命周期

静态生命周期```’static```
其生命周期存活于整个程序运行期间，所有的字符串字面值都拥有```static```生命周期
```let s:&'static str = "rust"```

##  4 闭包

##  5 智能指针

###  5.1 Box < T >

`Box<T>` 的主要特性是单一所有权，即同时只能有一个人拥有对其指向数据的所有权，并且同时只能存在一个可变引用或多个不可变引用，这一点与 Rust 中其他属于堆上的数据行为一致。

```RUST
fn main() {
    // b存储在栈上，5存储在堆上，b指向5的内存
    let b = Box::new(5);
    println!("b = {}",b);
}
```

单一所有权演示

```rust
fn main() {
    let a = Box::new(10);
    let b = a;
    // error[E0382]: use of moved value: `a`
    // 此行编译无法通过
    let c = a;
}
```

###  5.2 Deref  trait

实现Deref trait 允许我们重载解引用运算符

```rust
fn main() {
    let x = 5;
    let y = &x;
    assert_eq!(5, x);
    // 解引用
    assert_eq!(5, *y);
    
    let z = Box::new(x);
    assert_eq!(5,*z)
}
```



```rust
// 实现自定义的Box
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
// 实现解引用的trait
impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(5);
    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

解引用多态

###  5.3 Drop trait

类似于其它语言中的析构函数，当值离开作用域时执行此函数

```rust
struct Dog {
    name: String,
}
// 当变量离开作用域的时候自动调用
impl Drop for Dog {
    fn drop(&mut self) {
        println!("dog {} leave", self.name)
    }
}

fn main() {
    let a = Dog { name: "wangcai".to_string() };
    {
        let b = Dog { name: "dahuang".to_string() };
        println!("0 +++++++++++++++++++++++++")
    }
    println!("1 +++++++++++++++++++++++++")
}
```

提前释放的场景

```rust
struct Dog {
    name: String,
}

impl Drop for Dog {
    fn drop(&mut self) {
        println!("dog {} leave", self.name)
    }
}

// Rust提供了std::mem::drop()，可以提前释放内存
fn main() {
    let a = Dog { name: "wangcai".to_string() };
    let b = Dog { name: "dahuang".to_string() };
    drop(b);
    println!("0 +++++++++++++++++++++++++");
    println!("1 +++++++++++++++++++++++++");
}
```

###  5.4 Rc< T > &  Arc< T >

```Rc<T>``` 主要用于同一堆上所分配的数据区域需要有多个只读访问的情况，比起使用 ```Box<T>``` 然后创建多个不可变引用的方法更优雅也更直观一些，以及比起单一所有权，```Rc<T>``` 支持多所有权。

```Rc``` 为 Reference Counter 的缩写，即为引用计数，Rust 的 Runtime 会实时记录一个 ```Rc<T>``` 当前被引用的次数，并在引用计数归零时对数据进行释放（类似 Python 的 GC 机制）。因为需要维护一个记录 ```Rc<T>``` 类型被引用的次数，所以这个实现需要 Runtime Cost。

```rust
use std::rc::Rc;
fn main() {
    let a  = Rc::new(2);
    println!("count after creating a = {}",Rc::strong_count(&a));
    let b = Rc::clone(&a);
    println!("count after creating b = {}",Rc::strong_count(&a));
    {
        let c = Rc::clone(&a);
        println!("count after creating c = {}",Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}",Rc::strong_count(&a));
}

// count after creating a = 1
// count after creating b = 2
// count after creating c = 3
// count after c goes out of scope = 2
```

**注意：**

**```Rc<T>```是完全不可变的，可以理解为对同一块内存区域的多个只读指针**；

```rust
// 共享链表的情况
fn main() {
    let a = Node {
        value: 5,
        next: Box::new(None),
    };

    let b = Node{
        value:10,
        next:Box::new(Some(a)),
    };
    // error[E0382]: use of moved value: `a`
    // 编译无法通过
    // let c = Node{
    //     value:10,
    //     next:Box::new(Some(a)),
    // };

}

struct Node {
    value: u32,
    next: Box<Option<Node>>,
}


```

```rust
use std::rc::Rc;

// 使用Rc<T>改写，可顺利通过编译
fn main() {
    let a = Rc::new(Some(Node {
        value: 10,
        next: Rc::new(None),
    }));
    let b = Rc::new(&a);
    let c = Rc::new(&a);
}

struct Node {
    value: u32,
    next: Rc<Option<Node>>,
}
```

**`Rc<T>` 是只适用于单线程内的，尽管从概念上讲不同线程间的只读指针是完全安全的，但由于 `Rc<T>` 没有实现在多个线程间保证计数一致性，所以多线程情况下使用会报错。**

```rust
// 错误用法:多线程下使用Rc<T>
use std::thread;
use std::rc::Rc;

fn main() {
    let a = Rc::new(1);
    thread::spawn(|| {
        // Error: `std::rc::Rc<i32>` cannot be shared between threads safely
        let b = Rc::clone(&a);
    }).join();
}
```

```rust
// 使用Arc<T>代替 Atomic reference counter
use std::thread;
use std::sync::Arc;
 
fn main() {
    let a = Arc::new(1);
    thread::spawn(move || {
        let b = Arc::clone(&a);
        println!("{}", b);  // Output: 1
    }).join();
}
```

###  5.5 Cell < T >

```Cell<T>``` 其实和 ```Box<T>``` 很像，但后者同时不允许存在多个对其的可变引用，如果我们真的很想做这样的操作，在需要的时候随时改变其内部的数据，而不去考虑 Rust 中的不可变引用约束，就可以使用 ```Cell<T>```。```Cell<T>```允许多个共享引用对其内部值进行更改，实现了**「内部可变性」**

```rust
use std::cell::Cell;

fn main() {
    let a = Cell::new(1);
    let b = &a;
    let c = &a;
    a.set(2);
    b.set(3);
    c.set(4);
    // output:4
    println!("{}",a.get())
}
```

###  5.6 RefCell< T >











