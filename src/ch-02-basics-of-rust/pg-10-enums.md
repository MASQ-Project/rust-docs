## Enums

- Enums is the short form of enumerations.
- It allows us to define a type with possible values, these possible values are called _variants_.
- We can enumerate all possible variants, which is where enumeration gets its name.
- The total size that enum will allocate for it’s variant will be equal to the memory allocation of it’s largest variant. It works similar to unions in C.

#### Where to use Enums?

- When their are certain possible values for a type and those possible values may not coincide together.
- For Example, we can make an `enum` for Day, with possible variants Monday-Sunday, now for a certain day any two possible values will never coincide.
- Another Example, IP Address, it's possible variants will be IPV4, IPV6, for a certain IP address, it can only be either of the two.

- Here's an example definition:

  ```rust
  enum IpAddrKind {
      V4,
      V6,
  }
  ```

- To create an instance of ane enum, we use `::` operator:

  ```rust
  let four = IpAddrKind::V4;
  let six = IpAddrKind::V6;
  ```

- To use it in a function:

  ```rust
  // In fn declaration
  fn route(ip_kind: IpAddrKind) {}

  // In fn call
  route(IpAddrKind::V4);
  route(IpAddrKind::V6);
  ```

- Using Enums with Structs:

  ```rust
      enum IpAddrKind {
          V4,
          V6,
      }

      struct IpAddr {
          kind: IpAddrKind,
          address: String,
      }

      let home = IpAddr {
          kind: IpAddrKind::V4,
          address: String::from("127.0.0.1"),
      };

      let loopback = IpAddr {
          kind: IpAddrKind::V6,
          address: String::from("::1"),
      };
  ```

- Enums with associated data types:

  ```rust
  // Now, we don't need an extra struct
  enum IpAddr {
      V4(String),
      V6(String),
  }

  // We get a default constructor function for each variant
  let home = IpAddr::V4(String::from("127.0.0.1"));

  let loopback = IpAddr::V6(String::from("::1"));
  ```

- Defining enum variants with different data types:

```rust
enum IpAddr {
    // Defining variants with two different data types
    // is only possible through enums and not through enums with struct
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

- This is how [standard library defines IP addresses](https://doc.rust-lang.org/std/net/enum.IpAddr.html):

  ```rust
  struct Ipv4Addr {
      // --snip--
  }

  struct Ipv6Addr {
      // --snip--
  }

  // It is posible to put any data type inside
  // the enum variant, int, String, struct,
  // or even enum
  enum IpAddr {
      V4(Ipv4Addr),
      V6(Ipv6Addr),
  }

  ```

- Enum with complicated data types:

  ```rust
  // Cleaner Approach
  enum Message {
      Quit, // No data associated with it at all!
      Move { x: i32, y: i32 }, // Has named fields like struct
      Write(String),
      ChangeColor(i32, i32, i32),
  }

  // Uglier approach using struct
  struct QuitMessage; // unit struct
  struct MoveMessage {
      x: i32,
      y: i32,
  }
  struct WriteMessage(String); // tuple struct
  struct ChangeColorMessage(i32, i32, i32); // tuple struct
  ```

- It is possible to define associated functions on enums using `impl`:

  ```rust
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }

  impl Message {
      fn call(&self) {
          // method body would be defined here
      }
  }

  let _quit_message = Message::Quit; // We won't use parantheses because it is of Unit Type
  let _write_message = Message::Write(String::from("Hello")); // Constructor function, that accepts String and will stroe it on heap
  let _change_color_message = Message::ChangeColor(12, 12, 12); // Constructor function, that accepts three i32 values
  let _move_message = Message::Move {x: 5, y: 6}; // Works similar to creating new instance of struct with named fields

  _quit_message.call();
  ```

#### The `Option` Enum

- The Option type is used in many places because it encodes the very common scenario in which a value could be something or it could be nothing.
- Expressing this concept in terms of the type system means the compiler can check whether you’ve handled all the cases you should be handling.
- This functionality can prevent bugs that are extremely common in other programming languages.
- Rust doesn't have `Null`, so it uses `Option` enum with variants `Some` and `None`.
- This makes Rust extremely cool, you may read more about ["Null References: The Billion Dollar Mistake"](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values).
- The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind.
- Rust's `Option` enum will always ask you to offer solution for both `Some` and `None`.

  ```rust
  // It is generic over any data type T
  enum Option<T> {
      None,
      Some(T),
  }

  // Rust automatically inferred to be of type Option<i32> because we passed a number and i32 is it's default type
  let some_number = Some(5);
  // Similarly, Rust inferred Option<&str>, since we passed string literal
  let some_string = Some("a string");

  // Here, since None can belong to any data type, we explicitly define i32
  let absent_number: Option<i32> = None;
  ```

#### Why is having `Option<T>` any better than having `null`?

- In short, because `Option<T>` and `T` (where `T` can be any type) are different types, the compiler won’t let us use an `Option<T>` value as if it were definitely a valid value.
- For example, this code won’t compile because it’s trying to add an `i8` to an `Option<i8>`:

  ```rust
  let x: i8 = 5;
  let y: Option<i8> = Some(5);

  let sum = x + y;
  ```

- When we have a value of a type like `i8` in Rust, the compiler will ensure that we always have a valid value.
- We can proceed confidently without having to check for null before using that value.
- when we have an `Option<i8>`, we'll have to worry about possibly not having a value, and the compiler will make sure we handle that case before using the value.
- In other words, you have to convert an `Option<T>` to a `T` before you can perform `T` operations with it.
- Generally, this helps catch one of the most common issues with null: assuming that something isn’t null when it actually is.

- In languages like C, this will work and print something, even though we know it doesn't contain any value.

  ```c
  #include <stdio.h>

  int main() {
      int x;
      printf("Value of x: %i", x);

      return 0;
  }
  ```

- In Rust, it'll not compile, since it identifies an absence of value.

  ```rust
  fn main() {
      let number: i32;
      println!("Value of x: {}", number);
  }
  ```

- Everywhere that a value has a type that isn’t an `Option<T>`, you can safely assume that the value isn’t `null`.
- This was a deliberate design decision for Rust to limit null’s pervasiveness and increase the safety of Rust code.

