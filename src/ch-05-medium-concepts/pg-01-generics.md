### Generics

- Generics are used to prevent the duplication of **concepts** and are generalized over a type.
- Some examples of generics are `Result<T,E>`, `Option<T>`, `Vec<T>`, and `HashMap<K,V>`.
- Possible Use Cases:
  - You can define an enum or struct which can accomodate different data types.
  - You can define a function which can provide same functionality for different types. For Example, finding the largest element inside a vector of numbers or chars.
- In Rust, declaring generics aren't any slower than using concrete types, because it uses a process called _Monomorphization_ to achieve that. Monomorphization is the process of turning generic code into specific code by filling in the concrete types that are used when compiled.

#### Generics on structs

- To create a generic struct:

  ```rust
  // We used type T to make the struct generic
  // so that it can accomodate any type
  struct Point<T> {
      x: T,
      y: T,
  }

  fn main() {
      let integer = Point { x: 5, y: 10 };
      let float = Point { x: 1.0, y: 4.0 };

      // FAIL: First is i32 and the other is f32, hence different types.
      let wont_work = Point { x: 5, y: 4.0 };
  }
  ```

- To make the `wont_work` to work fine, we'll need to change the code as follows:

  ```rust
  struct Point<T, U> {
      x: T,
      y: U,
  }

  fn main() {
      let integer = Point { x: 5, y: 10 };
      let float = Point { x: 1.0, y: 4.0 };

      let will_work = Point { x: 5, y: 4.0 };
  }
  ```

#### Generics on Enums

- The `Option<T>` enum:

  ```rust
  enum Option<T> {
    Some(T),
    None
  }
  ```

- The `Result<T, E>` enum uses two types:

  ```rust
  enum Result<T, E> {
      Ok(T),
      Err(E),
  }
  ```

#### Generics on Functions

- To use generics on `impl` blocks:

  ```rust
  struct Point<T> {
      x: T,
      y: T,
  }

  // By using T after impl means that
  impl<T> Point<T> {
      fn x(&self) -> &T {
          &self.x
      }
  }

  // impl for just one concrete type
  impl Point<f32> {
      fn distance_from_origin(&self) -> f32 {
          (self.x.powi(2) + self.y.powi(2)).sqrt()
      }
  }

  fn main() {
      let p = Point { x: 5, y: 10 };

      println!("p.x = {}", p.x());
  }
  ```

- To use on `impl` on different types:

  ```rust
  struct Point<X1, Y1> {
      x: X1,
      y: Y1,
  }

  impl<X1, Y1> Point<X1, Y1> {
      fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
          Point {
              x: self.x,
              y: other.y,
          }
      }
  }

  fn main() {
      let p1 = Point { x: 5, y: 10.4 };
      let p2 = Point { x: "Hello", y: 'c' };

      let p3 = p1.mixup(p2);

      println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
  }
  ```
