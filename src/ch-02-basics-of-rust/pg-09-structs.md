## Structs

- A _struct_, or structure, is a **custom data type** that lets you name and package together multiple related values that make up a meaningful group.
- It is used to just define the data attributes as we do in Object Oriented Programming Languages.
- There are three types of Structs:
  1. Structs with Named Fields
  2. Tuple Structs
  3. Unit Structs

### Associated Functions and Methods

- Functions defined for structs using the `impl` keyowrd are called _associated functions_.
- The associated functions which accepts `self` as it's first argument are called _methods_.

#### Structs with Named Fields

- In structs, we name each piece of data, so it's clear what they mean. This name and data type pair are called _fields_.

- Struct definition:

  ```rust
  struct User {
      active: bool, // A Field
      username: String,
      email: String,
      sign_in_count: u64,
  }
  ```

- Creating a struct's instance:

  ```rust
  // If you specify mut, all the values will be mutable otherwise none
  let mut user1 = User {
      email: String::from("someone@example.com"),
      username: String::from("someusername123"),
      active: true,
      sign_in_count: 1,
  };
  ```

- Taking out and updating the values:

  ```rust
  user1.email = String::from("anotheremail@example.com");
  ```

- Defining functions for structs

  ```rust
  fn build_user(email: String, username: String) -> User {
      User {
          email, //We can write like this aslo-> email: email
          username,
          active: true,
          sign_in_count: 1,
      }
  }
  ```

- The struct update syntax (`..`), or spread operator in JS:

  ```rust
  // Initially
  let user2 = User {
      active: user1.active,
      username: user1.username,
      email: String::from("another@example.com"),
      sign_in_count: user1.sign_in_count,
  };

  // After using the struct update syntax (..)
  let user2 = User {
      email: String::from("another@example.com"),
      ..user1
  };
  ```

  Note: This update syntax, works same as assignment operator `=`, so stack values will get copied and heap values will be moved. Since, username is a String, it's value will be moved from `user1` to `user2`, hence `user1` can't be used again.

- To prevent this problem of ownership transfer, we can use `&str` instead of `String` but when we use references in structs, it won't actually compile but will ask for lifetimes.

  ```rust
  // FAIL: Lifetime specifier not provided.
  struct User {
      username: &str,
      email: &str,
      sign_in_count: u64,
      active: bool,
  }

  fn main() {
      let user1 = User {
          email: "someone@example.com",
          username: "someusername123",
          active: true,
          sign_in_count: 1,
      };
  }
  ```

- In this situation the compiler situation looks something like this:

  ```zsh
   --> src/main.rs:2:15
    |
  2 |     username: &str,
    |               ^ expected named lifetime parameter
    |
  help: consider introducing a named lifetime parameter
    |
  1 | struct User<'a> {
  2 |     username: &'a str,
    |
  ```

#### Tuple Structs

- Using Tuple Structs without Named Fields to Create Different Types:

  ```rust
  struct Color(i32, i32, i32);
  struct Point(i32, i32, i32);

  let black = Color(0, 0, 0);
  let origin = Point(0, 0, 0);
  ```

- To access their types, we use the `.` operator followed by the number of this argumnet.

  ```rust
  let color = Color(10, 25, 16);
  let red = color.0;
  let green = color.1;
  let blue = color.2;
  ```

#### Unit Structs

- They are structs without Any Fields (they act like `()`).
- They are Useful when we want to implement a trait on some type but don’t have any data that you want to store in the type itself.

  ```rust
  struct AlwaysEqual;

  let subject = AlwaysEqual;
  ```

#### Why do we use Structs?

1. It is a more sensible design choice to pass as minimum arguments as possible inside a function. For Example, if we need to calculate the area of rectangle, instead of passing `height` and `width`, it would be cleaner to pass the whole rectangle.
2. Now, this can be done with the tuples too. For Example, `let rect1 = (50, 30);` but the problem with this syntax is that any developer can confuse which one is width or height.
3. To make this process clearer and cleaner, we use `struct`, so that we can combine the data and still keep the meaning of each attribute intact.

   ```rust
   struct Rectangle {
       width: u32,
       height: u32,
   }

   fn main() {
       let rect1 = Rectangle {
           width: 50,
           height: 30
       };

       println!("The area of the rectangle is {} square pixels", area(&rect1));
   }


   // Passing Rectangle as a reference is important so that main fn
   // can retain it's ownership after this function is called.
   fn area(rectangle: &Rectangle) -> u32 {
       rectangle.width * rectangle.height
   }
   ```

#### Printing Variables

- Ways to Print the variables:

  - `{}` - Used to print variables with Display trait, for simple data types like int, string etc. we don't need to derive this attribute.
  - `{:?}` - Used to print complex variables with Debug trait, preferred for complex data type like struct, and we need to derive the `Debug` attribute.
  - `{:#?}` - Works similarly like `{:?}`, except it's preferred for structs with large number of fields.
  - `dbg!()` - It is a macro used with Debug trait to print the variables, file and line number. It prints to `stderr` instead of `stdout` (which `println!()` uses). It takes ownership, so prefer sending references to it.

- Here's an example of using the `dbg!()` macro:

  ```rust
  #[derive(Debug)]
  struct Rectangle {
      width: u32,
      height: u32,
  }

  fn main() {
      let scale = 2;
      let rect1 = Rectangle {
          width: dbg!(30 * scale), // It'll resolve the expression `30 * scale`, as if dbg!() call was never there, it happens due to ownership transfer
          height: 50,
      };

      dbg!(&rect1); // To maintian the scope of rect1 in main() we sent only the reference.
  }
  ```

- The output looks like this:

  ```zsh
     Compiling rectangles v0.1.0 (file:///projects/rectangles)
      Finished dev [unoptimized + debuginfo] target(s) in 0.61s
       Running `target/debug/rectangles`
  [src/main.rs:10] 30 * scale = 60
  [src/main.rs:14] &rect1 = Rectangle {
      width: 60,
      height: 50,
  }
  ```

- You can read more about [Derivable Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) and [Attributes](https://doc.rust-lang.org/reference/attributes.html).

#### Structs with Method Syntax

- When functions are defined in the context of a struct, enum or trait they are called as _Methods_.
- The first parameter of a method is always `self`, which represents the instance.

  ```rust
  #[derive(Debug)]
  struct Rectangle {
      width: u32,
      height: u32,
  }

  // Everything inside the impl block is associated with Rectangle
  impl Rectangle {

      // &self is a short hand for self: `&self` (references are used to prevent mutation)
      // You can pass the following too:
      // self - Ownership of instance
      // &self - Reference to the instance {Currently Using}
      // &mut self - Mutable Reference to the instance
      fn area(&self) -> u32 {
          self.width * self.height
      }

      // It is possible to name methods same as fields of struct
      // Usually these methods are used as getters, to keep the fields private but provide read only accees using the methods
      fn width(&self) -> bool {
        self.width > 0
      }

      // This is how we pass anotherr instance of same struct to a method
      fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
      }
  }

  fn main() {
      let rect1 = Rectangle {
          width: 30,
          height: 50,
      };

      let rect2 = Rectangle {
        width: 15,
        height: 25,
      };

      println!(
          "The area of the rectangle is {} square pixels.",
          rect1.area()
      );

      // If we use rect1.width() - Rust unserstands it as method and
      // if we use rect1.width - Rust unserstands it as a field
      if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
      };

      // This is how we can pass second instance while calling a method on first instance
      println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
  }
  ```

Note: When you call a method with `object.something()`, Rust automatically adds in `&`, `&mut`, or `*` so object matches the signature of the method. In other words, the following are the same:

```rust
p1.distance(&p2);
(&p1).distance(&p2);
```

- It is possible to use different `impl` blocks, it is a valid syntax.

  ```rust
  impl Rectangle {
      fn area(&self) -> u32 {
          self.width * self.height
      }
  }

  impl Rectangle {
      fn can_hold(&self, other: &Rectangle) -> bool {
          self.width > other.width && self.height > other.height
      }
  }
  ```

#### Associated Functions

- All the functions defined under `impl` are associated functions.
- Methods are associated functions which has `self` as an argument and we use `.` operator to access it.
- It is possible to define associated functions without passing `self` as the **first** argument, these functions are accessed through `::` operator.
- Here's an example:

  ```rust
  // Calling a method, also an associated function
  instance.method(some_argument);

  // Calling an associated function, without self as the first argument, hence not a method
  String::from("Hello, World!");
  ```

- These associated functions are commonly used as constructors. Also, for the previous example of Rectangle, we can use it as follows:

  ```rust
  impl Rectangle {
      // With this associated function we can create a new instance of Rectangle
      // by passing one value instead of two, hence creating a square.
      fn square(size: u32) -> Rectangle {
          Rectangle {
              width: size,
              height: size,
          }
      }
  }
  ```

- It can be called like this:

  ```rust
  let sq = Rectangle::square(3);
  ```
