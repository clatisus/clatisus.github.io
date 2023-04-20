+++
title = "Rust Type Coercion and the Dot Operator Explained"
date = "2023-04-08"
description = "For Rust beginners (like me), it has always been a mystery why some types suddenly transform into one another, and why some method calls with mismatched parameter types succeed. Well, the magic behind this is actually called *Type Coercion*. In this post, we will explore various type coercions and then explain how the dot operator works by applying the type coercion concepts we have learned."

[taxonomies]
tags = ["Rust"]
+++

## What is a type coercion?

So, what exactly is type coercion and why should we care about it?

Type coercion is simply an implicit type conversion. You might have seen implicit type conversions in many other languages, such as `C++`. In `C++`, when you write `int a = 4 + 5.2;`, two type coercion occurs. One happens on the operator `+`, from `int` to `double`. Another takes place on the assignment `=`, from `double` to `int`.

In Rust, you can perform explicit type conversion using the type cast operator `as`. Any conversions allowed by coercion can also be explicitly performed by `as`, but not vice versa. You have to write `let a: i32 = ((4 as f64) + 5.2) as i32;` to achieve the equivalent of the above example, as type coercion does not work here.

Getting to know type coercion can enable you to write idiomatic Rust code and help you understand how other code works. Let's get started exploring different coercions and where they can occur.

## Exploring different coercions

There isn't a formal definition of all coercions yet, and the reason for this is well explained in this blog[^what-can-coerce-and-where-in-rust]. I will do my best to provide appropriate examples to help you understand what does each coercion means.

### Transitive coercions

**Rule**: if `A` -> `B` and `B` -> `C` is possible, then `A` -> `C` is also possible

Note that this is not fully supported yet, and is currently a best-effort feature.

### Reference downgrade coercions

**Rule**: `&mut T` -> `&T`

A mutable reference can downgrade to an immutable reference. Let's take a look at an example:
```rust
fn takes_shared_ref(_n: &i32) {}

fn main() {
    let mut a = 10;
    takes_shared_ref(&mut a);
}
```

The example is quite clear, and what it does behind the scenes is that the `takes_shared_ref(&mut a);` will be desugared to a re-borrow `takes_shared_ref(&*(&mut a));`.

One thing to note is that in the above example, even though the mutable reference is dropped upon re-borrow, **its lifetime does not ended**[^downgrade_mut_lifetime]. Let's take a look at another example:
```rust
fn downgrade_to_shared_ref(n: &i32) -> &i32 {
    n
}

fn main() {
    let mut a = 10;
    let b = downgrade_to_shared_ref(&mut a);
    let c = &a;
    dbg!(b, c);
}
```

This results in the following error:
```bash
error[E0502]: cannot borrow `a` as immutable because it is also borrowed as mutable
 --> src/main.rs:8:13
  |
7 |     let b = downgrade_to_shared_ref(&mut a);
  |                                     ------ mutable borrow occurs here
8 |     let c = &a;
  |             ^^ immutable borrow occurs here
9 |     dbg!(b, c);
  |          - mutable borrow later used here
```

So even though the mutable reference has been downgraded to an immutable reference, the borrow checker still considers its lifetime as not eded. This might seem strange, but it actually guarantees memory safety.

Consider the following code:
```rust
use std::sync::Mutex;

struct Struct {
    mutex: Mutex<String>
}

fn main() {
    let mut s = Struct {
        mutex: Mutex::new("string".to_owned())
    };
    let str_mut: &mut str = s.mutex.get_mut().unwrap();
    let str_shared: &str = str_mut; // if str_mut's lifetime end here, the code compiles

    *s.mutex.lock().unwrap() = "str_shared becomes a dangling pointer".to_owned();

    dbg!(str_shared);
}
```

This also results in the same error:
```bash
error[E0502]: cannot borrow `s.mutex` as immutable because it is also borrowed as mutable
  --> src/main.rs:14:6
   |
11 |     let str_mut: &mut str = s.mutex.get_mut().unwrap();
   |                             ----------------- mutable borrow occurs here
...
14 |     *s.mutex.lock().unwrap() = "str_shared becomes a dangling pointer".to_owned();
   |      ^^^^^^^^^^^^^^ immutable borrow occurs here
15 |
16 |     dbg!(str_shared);
   |          ---------- mutable borrow later used here
```

If `str_mut`'s lifetime ended when it was downgraded, the code would compile, and a dangling pointer would be introduced.

### Deref coercions

**Rule**:
- if `T` implements `Deref<Target=U>`
  - `&T` -> `&U`
  - `&mut T` -> `&U` (*transitive*)
- if `T` implements `DerefMut<Target=U>`
  - `&mut T` -> `&mut U`

Your type can opt in these coercions by implementing the `Deref` or `DerefMut` trait.
```rust
pub trait Deref {
    type Target: ?Sized;

    pub fn deref(&self) -> &Self::Target;
}

pub trait DerefMut: Deref {
    pub fn deref_mut(&mut self) -> &mut Self::Target;
}
```

Having deref coercions enables us to use containers transparently as the type they contain (e.g. smart pointers), here's an example for `Box<String>`.
```rust
fn count(s: &str) -> usize {
    s.len()
}

fn main() {
    let a: Box<String> = Box::new("hello, world".to_string());
    println!("{}", count(&a))
}
```

The simple example above contains two deref coercions:
- `&Box<String>` -> `&String` (`Box<String>` implements `Deref<Target=String>`)
- `&String` -> `&str` (`String` implements `Deref<Target=str>`)

Another thing to note is that `&mut T` -> `&U` coercion calls `deref` instead of `deref_mut` (you might think of `&mut T` --*deref_mut*--> `&mut U` --*downgrade*--> `&U`, which is not the case), for example:
```rust
use std::ops::{Deref, DerefMut};

struct My(i32);

impl Deref for My {
    type Target = i32;
    fn deref(&self) -> &Self::Target {
        Box::leak(Box::new(self.0 + 1))
    }
}

impl DerefMut for My {
    fn deref_mut(&mut self) -> &mut Self::Target {
        Box::leak(Box::new(self.0 + 10))
    }
}

fn use_ref(a: &i32) {
    println!("use_ref {}", a);
}

fn main() {
    use_ref(&mut My(10)); // prints `use_ref 11`
}
```

### Raw pointer coercions

**Rule**: `*mut T` -> `*const T`

Similar to [Reference downgrade coercions](#reference-downgrade-coercions), pointers can also be downgraded.

### Reference & raw pointer coercions

**Rule**:
- `&T` -> `*const T`
- `&mut T` -> `*mut T`

This is useful when you want to pass a reference to a C function.

### Function pointer coercions

**Rule**: Closures without any captured variables can coerce to function pointers.

Example:
```rust
fn call_with(f: fn(i32) -> i32, a: i32) {
    println!("{}", f(a))
}

fn main() {
    call_with(|x| x * 2, 10)
}
```

### Subtype coercions

**Rule**: `T` -> `U` if `T` is a [subtype](https://doc.rust-lang.org/reference/subtyping.html) of `U`

Besides *parametric* polymorphism, Rust also supports *subtype* polymorphism, but only for lifetimes (maybe it should be called *subtime*).

The gist of subtyping in Rust is that if `'a` outlives `'b`, then `&'a T` is a subtype of `&'b T` (the longer-lived lifetime is the subtype, and the shorter-lived one is the super type).

The coercions means that lifetimes can be *shortened* at coercion sites, for example:
```rust
fn shorten_lifetime<'a>(a: &'a str) {
    println!("{}", a)
}

fn lengthen_lifetime(a: &'static str) {
    println!("{}", a)
}

fn main() {
    shorten_lifetime("Static");

    // let local = "local".to_string();
    // lengthen_lifetime(&local)
}
```

If we uncomment the last two lines, results in error:
```bash
error[E0597]: `local` does not live long enough
  --> src/main.rs:13:23
   |
13 |     lengthen_lifetime(&local)
   |     ------------------^^^^^^-
   |     |                 |
   |     |                 borrowed value does not live long enough
   |     argument requires that `local` is borrowed for `'static`
14 | }
   | - `local` dropped here while still borrowed
```

To support both parametric and subtype polymorphism, we need to answer how *generic types' subtyping relationships* relate to *the subtyping relationships of their generic parameters*. Well, the answer is called *variance*.

- **Covariance**: for some type `A<T>`, if `T` is a subtype of `U`, `A<T>` is a subtype of `A<U>`.
```rust
fn covariance<'a, 'b, T>(a: Vec<&'a T>) -> Vec<&'b T>
where
    // 'a is a subtype of 'b
    // then Vec<&'a T> is a subtype of Vec<&'b T>
    'a: 'b,
{
    a
}
```
- **Contravariance**: for some type `A<T>`, if `T` is a subtype of `U`, `A<U>` is a subtype of `A<T>`.
```rust
fn contravariance<'a, 'b, T>(a: fn(&'b T) -> ()) -> fn(&'a T) -> ()
where
    // 'a is a subtype of 'b
    // then fn(&'b T) -> () is a subtype of fn(&'a T) -> ()
    'a: 'b,
{
    a
}
```
- **Invariance**: for some type `A<T>`, no subtyping relationship exists between `A<T>` and any other type `A<U>`.

### Never coercions

**Rule**: `!` -> `T`

Never type can coerce into any type, this is especially useful when using macros like `unimplemented!`, `todo!`.

### Unsized coercions

#### Slice coercions

**Rule**: `[T; n]` -> `[T]`

Example:
```rust
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    let b: &[i32] = &a;
    println!("{:?}", b);
}
```

#### Trait object coercions

**Rule**: `T` -> `dyn U` (when `T` implements `U + Sized`, and `U` is [object safe](https://doc.rust-lang.org/reference/items/traits.html#object-safety))

Example:
```rust
trait Countable {
    fn count(&self) -> usize;
}

impl<T, const N: usize> Countable for [T; N] {
    fn count(&self) -> usize {
        self.len()
    }
}

impl<T> Countable for Vec<T> {
    fn count(&self) -> usize {
        self.len()
    }
}

fn print_count(ctb: &dyn Countable) {
    println!("{}", ctb.count());
}

fn main() {
    print_count(&[1, 2, 3]);
    print_count(&vec![1, 2, 3, 4]);
}
```

#### Trailing unsized coercions

**Rule**: `Foo<..., T, ...>` to `Foo<..., U, ...>`, when:
- `Foo` is a struct.
- `T` implements `Unsize<U>`.
- The last field of `Foo` has a type involving `T`.
- If that field has type `Bar<T>`, then `Bar<T>` implements `Unsized<Bar<U>>`.
- `T` is not part of the type of any other fields.

Example:
```rust
#[derive(Debug)]
struct Foo<T: ?Sized> {
    header: usize,
    value: T,
}

fn main() {
    let foo_array: Foo<[u8; 4]> = Foo {
        header: 0,
        value: [0, 1, 2, 3],
    };
    let foo_slice_ref: &Foo<[u8]> = &foo_array;
    println!("{foo_slice_ref:?}");
}
```

### Least upper bound coercions

In some cases, Rust needs to perform coercions for mutiple types at once. This coercion can be triggered by:

- A series of `if/else` branches.
- A series of `match` arms.
- A series of array elements.
- A series of `return`s in a closure.
- A series of `return`s in a function.

We say a set of types `T_0..T_n` are to be mutually coerced to some target type `T_t`. This is computed iteratively, the target type `T_t` begins as the type `T_0`. For each new type `T_i`, we do:
- If `T_i` can be coerced to the current target type `T_t`, then no change is made.
- Otherwise, check whether `T_t` can be coerced to `T_i`; 
  - if so, the `T_t` is changed to `T_i`. (This check is also conditioned on whether all of the source expressions considered thus far have implicit coercions.)
  - If not, try to compute a mutual supertype of `T_t` and `T_i`, which will become the new target type.

## Coercion sites

Coercions can happen in many places in Rust, here are some of them:

- **Variable declarations**: done with `let`, `const` or `static`. If type is annotated on the left hand side of the declaration, the compiler will check if the right hand side can be coerced to the type on the left hand side. 
```rust
let a: &str = &Box::new("abc".to_string()); // &Box<String> -> &String -> &str
let b: &[i32] = &[1, 2, 3, 4]; // &[i32; 4] -> &[i32]
```
- **Function parameters**: where the *actual parameter* is coerced into the type of the *formal parameter*. In method calls, the receiver type (`self`) is only able to use unsized coercions (explained at [The dot operator](#the-dot-operator)).
- **Function results**
```rust
use std::fmt::Display;
fn foo(x: &u32) -> &dyn Display {
    x
}
```
- **Struct/union/enum fields**: where the *actual type* is coerced into the *formal type* defined in the definition of the struct.
```rust
#[derive(Debug)]
struct Foo<'a> {
    x: &'a i8,
    y: &'a str,
}

fn main() {
    let y = &Box::new("abc".to_string());
    let a = Foo {
        x: &mut 42,
        y
    };
    println!("{:?}", a);
}
```
- **Unsized coercions only**: the followings are valid only for unsized coercions (where `U` can be obtained from `T` by [unsized coercions](#unsized-coercions))
  - `&T` -> `&U`
  - `&mut T` -> `&mut U`
  - `*const T` -> `*const U`
  - `*mut T` -> `*mut U`
  - `Box<T>` -> `Box<U>`

## The dot operator

There's a lot of magics behind the dot operator. It will perform auto-referencing, auto-dereferencing, and unsized coercions until it finds a method that matches the receiver type.

Suppose we have a function `foo` that has a receiver (a `self`, `&self` or `&mut self` parameter). Now, we call `value.foo()` where `value` is of type `T`. The compiler will try to find a method `foo` that matches the type `T` by performing the following steps:
- **By value**: try to match the receiver type `T` directly against the method receiver type. Note we are **NOT** trying to find a `T::foo` method here, we are trying to find a `foo` which has a receiver type `T`.
- **Autoref**: try to match the receiver type `&T` or `&mut T` (former has higher priority) against the method receiver type.
- If none of the above worked, it dereferences `T` and tries again.
  - if `T: Deref<Target=U>` then it tries again with type `U` instead of `T`
  - if it can't dereference `T`, it also try *unsizing* `T`

Let's try to understand this with an example:
```rust
let array: Rc<Box<[T; 3]>> = ...;
let first_entry = array[0];
```

How does the compiler actually compute `array[0]` when the array is behind so many indirections?
- First, `array[0]` is just sytax sugar for the [`Index`](https://doc.rust-lang.org/std/ops/trait.Index.html) trait, it will be desugared into `array.index(0)`.
```rust
pub trait Index<Idx>
where
    Idx: ?Sized,
{
    type Output: ?Sized;

    fn index(&self, index: Idx) -> &Self::Output;
}
```
- (*By value*) Then, the compiler checks if any method that matches the receiver type `Rc<Box<T; 3>>` exists. It doesn't.
- (*Autoref*) Neither does `&Rc<Box<T; 3>>` or `&mut Rc<Box<T; 3>>`.
- (*Deref*) the compiler dereferences `Rc<Box<T; 3>>` to `Box<T; 3>`.
- (*By value* & *Autoref*) `Box<T; 3>` and it's autorefs don't match any method receiver type.
- (*Deref*) the compiler dereferences `Box<T; 3>` to `[T; 3]`.
- (*By value* & *Autoref*) `[T; 3]` and it's autorefs don't match any method receiver type.
- (*Unsizing*) `[T; 3]` can't be dereferenced, so compiler unsizes it, giving `[T]`.
- Finally, the compiler find a method `index` that matches the receiver type `[T]` and calls it.

Let's take a look at another example[^nomicon-dot-operator]:
```rust
fn do_stuff<T: Clone>(value: &T) {
    let cloned = value.clone();
}
```
What's the type of `cloned`? Well, let's take a look at our candidate methods:
```rust
impl Clone for T { // indicated by the `T: Clone` bound
    fn clone(&self) -> Self;
}
impl Clone for &T { // standard library impl
    fn clone(&self) -> Self;
}
```
We can easily see that the first one is matched by value, so the type of `cloned` is `T`.

What would happen if the `T: Clone` bound is removed? Well, the first candidate will be removed, and the second one will be matched by autoref. So the type of `cloned` will be `&T`.

Note that the candidate method could be inherent (derived from the type of the receiver itself) or extension (derived from a trait that the receiver implements). And inherent methods have higher priority than extension methods. Let's take a look at this example:
```rust
struct MyStruct(i32);

impl MyStruct {
    fn hello(&mut self) -> () { // change to &self will print "inherent 10"
        println!("inherent {}", self.0)
    }
}

trait MyTrait {
    fn hello(&self) -> ();
}
impl MyTrait for MyStruct {
    fn hello(&self) -> () {
        println!("extension {}", self.0)
    }
}

fn main() {
    let a = MyStruct(10);
    a.hello(); // prints "extension 10"
}
```
The above code prints `trait 10` because the `&MyStruct` has higher priority than `&mut MyStruct` when it comes to method resolution. However, if we change the first `hello` method receiver type to `&self`, the compiler will print `impl 10` because inherent methods have higher priority than extension methods.

[^what-can-coerce-and-where-in-rust] Blog: [What Can Coerce, and Where, in Rust](https://www.possiblerust.com/guide/what-can-coerce-and-where-in-rust)

[^downgrade_mut_lifetime] Blog: [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#9-downgrading-mut-refs-to-shared-refs-is-safe)

[^nomicon-dot-operator] The Rustonomicon: [The Dot Operator](https://doc.rust-lang.org/nomicon/dot-operator.html)

[^reference-type-coercions] The Rust Reference: [Type coercions](https://doc.rust-lang.org/reference/type-coercions.html)

[^rfc-0401] Rust language [RFC 0401](https://rust-lang.github.io/rfcs/0401-coercions.html)

[^rfc-1558] Rust language [RFC 1558](https://rust-lang.github.io/rfcs/1558-closure-to-fn-coercion.html)

[^nomicon-coercions] The Rustonomicon: [Coercions](https://doc.rust-lang.org/nomicon/coercions.html)

[^effective-rust-understand-type-conversions] Effective Rust: [Understand type conversions](https://www.lurklurk.org/effective-rust/casts.html)
