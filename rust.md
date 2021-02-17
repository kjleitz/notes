# Rust

## Table of contents

[toc]

## Blocks & implicit returns from scope

The final expression in a block is implicitly returned if you do not terminate it as a _statement_ with a semicolon.

```rust
fn main() {
  let the_length = get_length(String::from("yeah"));
  if the_length == 4 {
    println!("yay!");
  } else {
    println!("whaaaaaaa");
  }
}

fn get_length(some_string: String) -> usize {
  some_string.len()
}
```

```rust
fn main() {
  // yeah you can do anonymous blocks like this
  let the_length = {
    let some_string = String::from("yeah");
    get_length(some_string)
  };

  if the_length == 4 {
    println!("yay!");
  } else {
    println!("whaaaaaaa");
  }
}

fn get_length(some_string: String) -> usize {
  some_string.len()
}
```

```rust
fn main() {
  let the_length = {
    let some_string = String::from("yeah");
    get_length(some_string)
  };

  let it_is_four = if the_length == 4 {
    true
  } else {
    false
  };

  if it_is_four {
    println!("yay!");
  } else {
    println!("whaaaaaaa");
  }
}

fn get_length(some_string: String) -> usize {
  some_string.len()
}
```

## Ownership

Once a variable (one that's a reference to memory allocated on the heap, as opposed to a fixed-size primitive which would be put on the stack) is taken out of the scope that owns it, it's dropped from memory:

```rust
fn main() {
    let foo = "hey";
    let bar = foo;
    println!("foo: {}", foo);
    println!("bar: {}", bar);

    let baz = String::from(foo);
    let bam = baz;
    // println!("baz: {}", baz); //=> error: it's been "move"d! no longer valid
    println!("bam: {}", bam);

    let crap = String::new();
    takes_ownership(crap);
    // println!("crap: {}", crap); //=> error: it's been "move"d! no longer valid

    let blah = 5;
    does_not_take_ownership(blah);
    println!("blah: {}", blah);
}

fn takes_ownership(some_string_reference: String) {
    println!("(takes_ownership) some_string_reference: {}", some_string_reference);
}

fn does_not_take_ownership(some_simple_scalar: i32) {
    println!("(does_not_take_ownership) some_simple_scalar: {}", some_simple_scalar);
}
```

(TODO: add more here)

## Structs

### Definition

```rs
struct MultipleChoiceQuestion {
    answer: i32,
    choices: [i32; 3],
}
```

### Associated functions/methods

```rs
impl MultipleChoiceQuestion {
    fn new(answer: i32, choices: [i32; 3]) -> Self {
        Self {
            answer: answer,
            choices: choices,
        }
    }

    fn answer_valid(&self) -> bool {
        self.choices.contains(&self.answer)
    }

    fn choose_answer(&mut self, answer_index: usize) {
        self.answer = self.choices[answer_index];
    }
}

fn main() {
    // ...
    let mut mult = MultipleChoiceQuestion::new(2, [1, 2, 3]);
    println!("{}", mult.answer);         //=> 2
    println!("{}", mult.answer_valid()); //=> true
    mult.choose_answer(0);               // ...
    println!("{}", mult.answer);         //=> 1
    println!("{}", mult.answer_valid()); //=> true
    mult.choose_answer(10);              // PANIC!
    println!("{}", mult.answer);
    println!("{}", mult.answer_valid());
    // ...
}
```

If the struct is generic you have to add the type parameters to the `impl` as well:

```rs
struct Foo<T> {
  bar: T,
}

impl<T> Foo<T> {
  fn get_bar(&self) -> &T {
    &self.bar
  }
}
```

And constraints on those type parameters also have to be in the `impl` (but not necessarily on the `struct`; if the constraints don't match, though, you can't use the method defined in the `impl`):

```rs
struct Foo<T> {
    bar: T,
}

impl<T> Foo<T>
where T: std::fmt::Debug {
    fn get_bar(&self) -> &T {
        println!("{:?}", self.bar);
        &self.bar
    }
}

fn main() {
    let mut mult = MultipleChoiceQuestion::new(2, [1, 2, 3]);

    let blahblah = Foo {
        bar: mult,
    };

    let blahblahbar = blahblah.get_bar(); // ERROR! no method `get_bar` for this variant because `MultipleChoiceQuestion` doesn't have the `std::fmt::Debug` trait
}
```

...vs:

```rs
struct Foo<T>
where T: std::fmt::Debug {
    bar: T,
}

impl<T> Foo<T>
where T: std::fmt::Debug {
    fn get_bar(&self) -> &T {
        println!("{:?}", self.bar);
        &self.bar
    }
}

fn main() {
    let mut mult = MultipleChoiceQuestion::new(2, [1, 2, 3]);

    let blahblah = Foo {
        bar: mult,
    }; // ERROR! `MultipleChoiceQuestion` doesn't implement `std::fmt::Debug`

    let blahblahbar = blahblah.get_bar();
}
```

...vs:

```rs
#[derive(Debug)]
struct MultipleChoiceQuestion {
  // ...
}

struct Foo<T>
where T: std::fmt::Debug {
    bar: T,
}

impl<T> Foo<T>
where T: std::fmt::Debug {
    fn get_bar(&self) -> &T {
        println!("{:?}", self.bar);
        &self.bar
    }
}

fn main() {
    let mut mult = MultipleChoiceQuestion::new(2, [1, 2, 3]);

    let blahblah = Foo {
        bar: mult,
    };

    let blahblahbar = blahblah.get_bar();
    // nice!
}
```

### Traits 

https://youtu.be/gi0AQ78diSA

```rs
struct FreeFormQuestion {
    answer: i32,
}

trait QuestionTrait {
    fn answer_index_valid(&self, index: usize) -> bool;
}

impl QuestionTrait for MultipleChoiceQuestion {
    fn answer_index_valid(&self, index: usize) -> bool {
        let min = 0;
        let max = self.choices.len() - 1;
        min <= index && index <= max
    }
}

impl QuestionTrait for FreeFormQuestion {
    fn answer_index_valid(&self, _index: usize) -> bool {
        false
    }
}

fn answer_index_valid_on_some_question_stuff(index: usize, question: &dyn QuestionTrait) -> bool {
    question.answer_index_valid(index)
}

fn main() {
    // ...
    println!("{}", mult.answer_index_valid(0)); //=> true
    println!("{}", mult.answer_index_valid(1)); //=> true
    println!("{}", mult.answer_index_valid(2)); //=> true
    println!("{}", mult.answer_index_valid(3)); //=> false

    println!("{}", answer_index_valid_on_some_question_stuff(0, &mult)); //=> true
    println!("{}", answer_index_valid_on_some_question_stuff(1, &mult)); //=> true
    println!("{}", answer_index_valid_on_some_question_stuff(2, &mult)); //=> true
    println!("{}", answer_index_valid_on_some_question_stuff(3, &mult)); //=> false

    let free_form = FreeFormQuestion {
      answer: 123,
    };

    println!("{}", free_form.answer_index_valid(0)); //=> false
    println!("{}", answer_index_valid_on_some_question_stuff(0, &free_form)); //=> false
    // ...
}
```

### Generics

Generic structs have inferrable type params (unlike generic interfaces in TypeScript), which is pretty cool

```rs
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    // ...
    let p1 = Point { x: 1, y: 2 };
    //=> Point<i32>
    let p2 = Point { x: 1.0, y: 2.5 };
    //=> Point<f64>
    // ...
}
```

## Enums

https://youtu.be/N28mGv1L8EM

TODO

### Generics

Enums can have generic type parameters, too

https://youtu.be/nvur2Ast8hE

TODO

## Functions

### Generics

also functions can have generic type parameters

https://youtu.be/nvur2Ast8hE

TODO

```rs
fn add1<T>(n1: T, n2: T) -> T {
  n1 + n2 // ERROR! T isn't necessarily add-able
}

// put a constraint on T (constraints can be traits)
fn add<T: std::ops::Add>(n1: T, n2: T) -> T {
  n1 + n2 // ERROR! doesn't know that adding two values of type T outputs another T
}

// put a better constraint on T
fn add<T: std::ops::Add<Output=T>>(n1: T, n2: T) -> T {
  n1 + n2 // nice
}

fn subtract<T: std::ops::Add<Output=T>>(n1: T, n2: T) -> T {
  n1 - n2 // ERROR! need to know that they're subtract-able
}

// intersect the two constraints with `+`
fn subtract<T: std::ops::Add<Output=T> + std::ops::Sub<Output=T>>(n1: T, n2: T) -> T {
  n1 - n2 // nice
}

// debug?
fn subtract<T: std::ops::Add<Output=T> + std::ops::Sub<Output=T> + std::fmt::Debug>(n1: T, n2: T) -> T {
  println!("n1 is {:?}", n1); // nice
  n1 - n2 // nice
}

// method signature is getting long, so...
fn subtract<T>(n1: T, n2: T) -> T
where T: std::ops::Add<Output=T> + std::ops::Sub<Output=T> + std::fmt::Debug {
  println!("n1 is {:?}", n1); // nice
  n1 - n2 // nice
}

// more contraints, multi-line
fn subtract<T, E>(n1: T, n2: T, reason: E) -> T
where T: std::ops::Add<Output=T> + std::ops::Sub<Output=T> + std::fmt::Debug,
      E: std::fmt::Debug {
  println!("reason is {:?}", reason); // nice
  println!("n1 is {:?}", n1); // nice
  n1 - n2 // nice
}
```

## Lifetimes

https://youtu.be/1QoT9fmPYr8

```rs
fn main() {
  let foo = 10;
  let result = some_fn(&foo);
  println!("{}", result);
}

// ERROR! the owner doesn't live long enough for the reference to return to the
// scope the function is being called from
fn some_fn() -> &i32 {
  let blah = 1;
  &blah
}

// nice! that reference is being passed in from the calling scope and it'll live
// long enough for sure
fn some_fn(param: &i32) -> &i32 {
  param
}

// explicitly defining the lifetime; this is the same exact thing as the above
// function (under the hood, it's done automatically)
fn some_fn<'a>(param: &'a i32) -> &'a i32 {
  param
}

// when the compiler sees a reference, it'll add a lifetime parameter for each
// one... as many as it needs to cover the references in the parameters, and
// each reference is given its OWN lifetime
fn some_fn<'a, 'b>(p1: &'a i32, p2: &'b f64, p3: Vec<i8>) -> &'a i32 {
  p1
}
```

### Lifetime subtyping and requirements

```rs
fn main() {
  let foo = 10;
  let bar = 20;
  let result = some_fn(&foo, &bar);
  println!("{}", result);
}

fn some_fn<'a, 'b>(p1: &'a i32, p2: &'b i32) -> &'a i32 {
  if p1 > p2 {
    p1
  } else {
    p2 // ERROR! mismatched lifetime; returns 'a but p2 is actually 'b
  }
}

// you can subtype 'b to tell the compiler it has to last at least as long as 'a
fn some_fn<'a, 'b: 'a>(p1: &'a i32, p2: &'b i32) -> &'a i32 {
  if p1 > p2 {
    p1
  } else {
    p2
  }
}

// when they all have the same lifetime, it doesn't enforce that the parameters
// all _actually_ have the same lifetime... it basically takes the minimum of
// the lifetimes given
fn some_fn<'a>(p1: &'a i32, p2: &'a i32) -> &'a i32 {
  if p1 > p2 {
    p1
  } else {
    p2
  }
}

// ERROR! these are actually different lifetimes... they either gotta be 'static
// or specify a common lifetime
fn some_fn(p1: &i32, p2: &i32) -> &i32 {
  if p1 > p2 {
    p1
  } else {
    p2
  }
}

// nice!
fn some_fn<'a>(p1: &'a i32, p2: &'a i32) -> &'a i32 {
  if p1 > p2 {
    p1
  } else {
    p2
  }
}

// but if we're not returning a reference, then it's not a problem at all
fn some_fn(param: &Vec<i8>) -> Vec<i8> {
  param.clone()
}

// likewise, if we have no reference inputs then it's also not a problem
fn some_fn(param: Vec<i8>) -> Vec<i8> {
  param.clone()
}

// if we never return a reference parameter back to the calling scope, then
// we're also fine and we don't have to worry about it... the default lifetime
// provided by the compiler is sufficient
fn some_fn(param: &i32) -> bool {
  param > &10
}

// also, if there's only one reference parameter and one reference output, the
// compiler will automatically enforce that the lifetimes are the same and you
// don't have to worry about it
fn some_fn(param: &i32) -> &i32 {
  param
}
```

### Combining lifetime parameters & type parameters

```rs
fn main() {
  let foo = 10;
  let bar = 20;
  let min1 = minimum(&foo, &bar);
  println!("{}", min1); //=> 10

  let baz = "bbb";
  let bam = "aaa";
  let min2 = minimum(&baz, &bam);
  println!("{}", min2); //=> "aaa"
}

// it's okay, they can be friends and sit next to each other:
fn minimum<'a, T: std::cmp::PartialOrd>(val1: &'a T, val2: &'a T) -> &'a T {
  if val1 < val2 {
    val1
  } else {
    val2
  }
}
```

### Lifetimes in structs

structs with reference data also have to have lifetimes specified:

```rs
struct Foo {
  bar: Vec<i32>,
  baz: &Vec<i32>, // ERROR! missing lifetime specifier
}

struct Foo<'a> {
  bar: Vec<i32>,
  baz: &'a Vec<i32>, // nice
}

struct Foo<'a> {
  bar: Vec<i32>,
  baz: &'a Vec<i32>, // nice
  baz: &'a Vec<i32>, // nice
}

struct Foo<'a, 'b> {
  bar: Vec<i32>,
  baz: &'a Vec<i32>, // nice
  baz: &'b Vec<i32>, // nice
}

struct Foo<'a, 'b: 'a> {
  bar: Vec<i32>,
  baz: &'a Vec<i32>, // nice
  baz: &'b Vec<i32>, // nice
}
```
