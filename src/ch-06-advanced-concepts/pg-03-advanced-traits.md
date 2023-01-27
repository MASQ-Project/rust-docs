### Advanced Traits

#### Associated Types

- This is how you can create an associate type inside a trait

  ```rust
  pub trait Iterator {
      type Item; // You see, this is the associated type

      fn next(&mut self) -> Option<Self::Item>; // Now, this trait can use this type in it's signatures
  }
  ```

- Here's an implementation of this trait on one of our types Counter:

  ```rust
  impl Iterator for Counter {
      type Item = u32;

      fn next(&mut self) -> Option<Self::Item> {
          // --snip--
  ```

- Now, you might be wondering, can't we do something like this with using traits with generics?

  ```rust
  pub trait Iterator<T> {
      fn next(&mut self) -> Option<T>;
  }
  ```

- Here are the differences between Associated Types and Traits with generics?

  | Traits with Associated Types                                                      | Traits with Generics                                                              |
  | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
  | There will be only one implementation for a type.                                 | There can be multiple implementations for a type, using individual concrete type. |
  | For Example, `impl Iterator for Counter`                                          | For Example, `impl Iterator<String> for Counter`, and many more.                  |
  | Using functions of these traits will not require you to provide type annotations. | You'll need to provide type annotations to provide which iteration to use.        |

#### Operator Overloading

- This is how you can add two types by using the `+` operator

  ```rust
  // This library `std::ops` contains all the overloadable operators
  use std::ops::Add;

  #[derive(Debug, Copy, Clone, PartialEq)]
  struct Point {
      x: i32,
      y: i32,
  }

  impl Add for Point {
      type Output = Point; // associated type will restrict the number of implementations

      fn add(self, other: Point) -> Point { // This fn will decide how the `+` will behave for the type Point
          Point {
              x: self.x + other.x,
              y: self.y + other.y,
          }
      }
  }

  fn main() {
      assert_eq!(
          Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
          Point { x: 3, y: 3 }
      );
  }
  ```

- This is how the trait `Add` is defined in the Rust's library `std::ops`

  ```rust
  // This syntax `<Rhs=Self>` is called "default parameters"
  // In case, someone implementing this trait doesn't define the type
  // then the type defined in default parameter will be used everywhere in this trait
  trait Add<Rhs=Self> {
      type Output;

      fn add(self, rhs: Rhs) -> Self::Output; // The "default parameter" is used here inside argument `rhs`
  }
  ```

- You can also customize `Rhs`, that'll mean you'll be adding a value of different type to the main type

  ```rust
  use std::ops::Add;

  struct Millimeters(u32);
  struct Meters(u32);

  impl Add<Meters> for Millimeters { // Modified Rhs to Meters, otherwise the default will be Self (or Millimeters)
      type Output = Millimeters;

      fn add(self, other: Meters) -> Millimeters {
          Millimeters(self.0 + (other.0 * 1000))
      }
  }
  ```

- You’ll use default type parameters in two main ways:
  - To extend a type without breaking existing code
  - To allow customization in specific cases most users won’t need

#### Function with same name in multiple traits

- Following things are allowed:

  - Multiple traits to have functions with same name
  - A type implementing all these traits

- Here's an implementation:

  ```rust
  trait Pilot {
      fn fly(&self);
  }

  trait Wizard {
      fn fly(&self);
  }

  struct Human;

  impl Pilot for Human {
      fn fly(&self) {
          println!("This is your captain speaking.");
      }
  }

  impl Wizard for Human {
      fn fly(&self) {
          println!("Up!");
      }
  }

  impl Human {
      fn fly(&self) {
          println!("*waving arms furiously*");
      }
  }

  fn main() {
      let person = Human;
      Pilot::fly(&person); // You'll have to pass the reference to the person, because it takes self as an argument
      Wizard::fly(&person); // if we had two types that both implement one trait, this self would recognize the correct type
      person.fly(); // The direct implementation will get called first, instead you can also use Human::fly(&person)
  }
  ```

- Let's see what happens if the _associated function_ of a _struct_ and a _trait_ has same name

  ```rust
  trait Animal {
      // Notice, there is no self passed inside,
      // multiple types can implement this trait,
      // Calling this function won't be direct,
      // as the trait won't be able to infer
      // which type's implementation to call
      fn baby_name() -> String;
  }

  struct Dog;

  impl Dog {
      // This is an associated function, which can be called
      // using `Dog::baby_name()`, just like any other
      // associated fn without self
      fn baby_name() -> String {
          String::from("Spot")
      }
  }

  impl Animal for Dog {
      fn baby_name() -> String {
          String::from("puppy")
      }
  }

  fn main() {
      println!("A baby dog is called a {}", Dog::baby_name()); // The direct implementation will get called
      println!("A baby dog is called a {}", Animal::baby_name()); // Won't work, this associated fn don't has self, remember? It can't infer the type
      println!("A baby dog is called a {}", <Dog as Animal>::baby_name()); // We need to tell Rust that we want to use the implementation of Animal for Dog
  }
  ```

- This is the fully qualified syntax, used for all associated functions (including methods)

  ```rust
  <Type as Trait>::function(receiver_if_method, next_arg, ...); // You don't need to provide all the information that Rust can infer
  ```

#### Supertraits

- Sometimes, you might need one trait to use another trait’s functionality.
- The traits you would be using to build your own trait is called _Supertrait_.
- One important thing is that, now your trait can only be implemented on the types that already implements the Supertrait.

- Here's an implementation. This `OutlinePrint` can **only** be implemented on any trait that implements `Display`.

  ```rust
  use std::fmt;

  trait OutlinePrint: fmt::Display { // It is also like bounding a trait with another trait
      fn outline_print(&self) {
          let output = self.to_string(); // It works only because all the types already implements `Display`
          let len = output.len();
          println!("{}", "*".repeat(len + 4));
          println!("*{}*", " ".repeat(len + 2));
          println!("* {} *", output);
          println!("*{}*", " ".repeat(len + 2));
          println!("{}", "*".repeat(len + 4));
      }
  }
  ```

- For reference, you can also implement `fmt::Display` on your type, like this:

  ```rust
  struct Point {
      x: i32,
      y: i32,
  }

  use std::fmt;

  impl fmt::Display for Point {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "({}, {})", self.x, self.y)
      }
  }
  ```

- Now, we know that `Point` implements `Display`, now we can implement `OutlinePrint` like this:

  ```rust
  impl OutlinePrint for Point {}
  ```

- The output will look like this for the fn `outline_print`

  ```zsh
  **********
  *        *
  * (1, 3) *
  *        *
  **********
  ```

#### Newtype Pattern

_The **Orphan Rule** states that we’re allowed to implement a trait on a type as long as either the trait or the type are local to our crate._

- The _newtype pattern_ , helps us to get around this restriction.
- All we need to do is to create a new type in a tuple struct.
- Let's say, we want to implement `Display` on `Vec<T>`. Both of them are not local to our code, thereby orphan rule will prevent us from implementing it.
- Now, we can use this newtype pattern for getting a workaround. What we'll be building is a wrapper that will make the `Vec<T>` local to us.

  ```rust
  use std::fmt;

  struct Wrapper(Vec<String>); // A new struct, which is just a wrapper for Vec<String>

  impl fmt::Display for Wrapper { // Now, we can implement Display on Wrapper, but we can't do directly on Vec<T>
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "[{}]", self.0.join(", ")) // self.0 is used to take the Vector out
      }
  }

  fn main() {
      // The only disadvantage is that, we'll need to create
      // a vector inside a wrapper to use the Display trait
      let w = Wrapper(vec![String::from("hello"), String::from("world")]);
      println!("w = {}", w);
  }
  ```

- If we wanted the new type to have every method the inner type has, [implementing the Deref trait](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait) on the Wrapper to return the inner type would be a solution.
- If we don’t want the Wrapper type to have all the methods of the inner type — for example, to restrict the Wrapper type’s behavior — we would have to implement just the methods we do want manually.
