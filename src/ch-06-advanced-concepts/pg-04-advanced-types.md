### Advanced Types

#### Uses of the Newtype Pattern

- _Type Safety_: We can wrap `u32` values with structs `Millimeters` and `Meters`. Now, if a value is stored in `Millimeters`, it is safe to say that this value can't call functions defined for `Meters`, and vice versa is true.
- _Abstraction_: We could provide a `People` type to wrap a `HashMap<i32, String>` that stores a person’s ID associated with their name. Code using `People` would only interact with the public API we provide, such as a method to add a name string to the People collection; that code **wouldn’t need to know** that we assign an `i32` ID to names internally.

#### Type Aliases

- This is how you can create a type alias:

  ```rust
  type Kilometers = i32;
  ```

- How it works?

  - Values with type `Kilometers` will be treated same as `i32`.
  - We aren't creating a new type, we're just adding a _synonym_ to `i32`, called `Kilometers`

- Hence, doing something like this is totally fine:

  ```rust
  let x: i32 = 5;
  let y: Kilometers = 5;

  println!("x + y = {}", x + y); // This will work
  ```

- The advantage is that it will give us the flexibility that any function with an argument of `i32`, we can pass `Kilometers` value to it.
- The disadvantage is that we don't get the type checks as we get in the newtype pattern.

- You can create an alias where you want to prevent naming a complex type.

  ```rust
  type Thunk = Box<dyn Fn() + Send + 'static>; // A type that stores closure

  let f: Thunk = Box::new(|| println!("hi")); // We don't need to specify the longer type, instead we can say just "Thunk"

  fn takes_long_type(f: Thunk) { // Again, we don't need to specify the longer type, just "Thunk"
    ...
  }
  ```

Fun Fact: Thunk is a word for code to be evaluated at a later time, so it’s an appropriate name for a closure that gets stored

- We can also shorten a `Result` values of I/O operations like this

  ```rust
  type Result<T> = std::result::Result<T, std::io::Error>;
  ```

- Now, `Result<usize, Error>` can be replaced with `Result<usize>` and `Result<(), Error>` can be replaced with `Result<()>`. Also, we can use the `?` operator, since it's the same type.

#### The never type `!`

- This type never returns. In type theory lingo it is known as the _empty type_ because it has no values.
- Rust prefers to call it the never type because it stands in the place of the return type when a function will never return.

  ```rust
  // People read it as, "the function bar(), returns never", and are called diverging functions
  fn bar() -> ! {
      // --snip--
  }
  ```

- Rust never allows a variable to have different possible data types.

  ```rust
  // FAIL: You can't create a variable "guess", that may have either number or string
  let guess = match guess.trim().parse() {
      Ok(_) => 5,
      Err(_) => "hello",
  };
  ```

- But this is possible:

  ```rust
  let guess: u32 = match guess.trim().parse() {
      Ok(num) => num,
      Err(_) => continue, // Wait a minute, how's this even allowed?
  };
  ```

- The `continue` has a never type (`!`), which means it'll never return any value. That is, when Rust computes the type of `guess`, it looks at both match arms, the former with a value of `u32` and the latter with a `!` value. Because `!` can never have a value, Rust decides that the type of guess is `u32`.

- You can remember it this way, the `!` can get coerced to any other type.

- This is the original implementation of `unwrap!()`. Here, the return type is coerced to a single type `T`, and that's because `panic!()` ends the program and has a never type (`!`).

  ```rust
  impl<T> Option<T> {
      pub fn unwrap(self) -> T {
          match self {
              Some(val) => val,
              None => panic!("called `Option::unwrap()` on a `None` value"),
          }
      }
  }
  ```

- The value of the expression of `loop` is also of the type `!`.

  ```rust
  print!("forever ");

  loop {
      print!("and ever ");
  }
  ```

#### Dynamically Sized Traits and the `Sized` trait

- _DSTs_ or _unsized_ types let us write code using values whose size can only be known at runtime.

- So, Rust doesn't let us create strings with `str` (not `&str`):

  ```rust
  let s1: str = "Hello there!";
  let s2: str = "How's it going?";
  ```

- Why's that?

  - Rust needs to know a fixed size of a type. Here `s1` takes 12 bytes and `s2` takes `15` bytes.
  - It's not possible to accomodate all the strings in a single fixed size.

- What's the solution?

  - `&str` is the solution.
  - It stores two values: the address of the `str` and its length.
  - So, that makes `&str` will only need two `usize`, one for the address and the other for the length.
  - That's why, we always know the size of a `&str`, no matter how long the string it refers to is.

- In general, this is the way in which dynamically sized types are used in Rust.
- The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a pointer of some kind.

- The traits can be Dynamically Sixed too. All we need to do is to put them behind a pointer, such as `&dyn Trait` or `Box<dyn Trait>`.

- The `Sized` trait

  - A trait that determines whether or not a type's size is known at compile time.
  - You may create a generic function like this:

    ```rust
    fn generic<T>(t: T) {
        // --snip--
    }
    ```

  - But, Rust treats it as if it was re-written like this:

    ```rust
    // This means generic functions will only work on
    // types who's size is known at the compile time
    fn generic<T: Sized>(t: T) {
        // --snip--
    }
    ```

  - It's possible to get over with this restriction:

    ```rust
    // The ?Sized means “T may or may not be Sized”
    // Now, this fn will accept T whose size may or may not be known at compile time
    // The ?Trait syntax with this meaning is only available for Sized, not any other traits.
    // Also, notice, we're using `&T` and not `T`, now we'll use `T` behind some kind of pointer, here it's reference
    fn generic<T: ?Sized>(t: &T) {
        // --snip--
    }
    ```
