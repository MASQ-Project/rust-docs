## Pattern Matching

- `match` Arms

  ```rust
  match VALUE {
      PATTERN => EXPRESSION,
      PATTERN => EXPRESSION,
      PATTERN => EXPRESSION,
  }
  ```

- Conditional `if let` statements

  ```rust
  fn main() {
      let favorite_color: Option<&str> = None;
      let is_tuesday = false;
      let age: Result<u8, _> = "34".parse();

      if let Some(color) = favorite_color {
          println!("Using your favorite color, {}, as the background", color);
      } else if is_tuesday {
          println!("Tuesday is green day!");
      } else if let Ok(age) = age {
          if age > 30 {
              println!("Using purple as the background color");
          } else {
              println!("Using orange as the background color");
          }
      } else {
          println!("Using blue as the background color");
      }
  }
  ```

- `while let` loops

  ```rust
      let mut stack = Vec::new();

      stack.push(1);
      stack.push(2);
      stack.push(3);

      while let Some(top) = stack.pop() {
          println!("{}", top);
      }
  ```

- `for` loops

  ```rust
      let v = vec!['a', 'b', 'c'];

      for (index, value) in v.iter().enumerate() {
          println!("{} is at index {}", value, index);
      }
  ```

- `let` statements

  ```rust
      let (x, y, z) = (1, 2, 3);
  ```

- function parameters

  ```rust
  fn print_coordinates(&(x, y): &(i32, i32)) {
      println!("Current location: ({}, {})", x, y);
  }

  fn main() {
      let point = (3, 5);
      print_coordinates(&point);
  }
  ```

#### Forms of Pattern

| Refutable                                                                                                                                | Irrefutable                                                                                                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Patterns that can fail to match for some possible value.                                                                                 | Patterns that'll match for any possible value.                                                                                                                          |
| For Example, in `if let Some(x) = a_value`, the `Some(x)` can fail to match if `a_value` is `None`.                                      | For Example, in `let x = 5;`, the `x` matches anything and therefore cannot fail to match.                                                                              |
| The `if let` and `while let` expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns. | Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match. |

- Using a refutable pattern where Rust requires an irrefutable pattern:

  ```rust
  // FAIL: The code doesn't know what to do with a None value
  // Some(x) is a refutable pattern
  // let requires an irrefutable pattern
  let Some(x) = some_option_value;
  ```

- Using irrefutable pattern where Rust requires a refutable pattern:

```rust
// WARN: It doesn’t make sense to use if let with an irrefutable pattern
// x is an irrefutable pattern
// if let is a refutable pattern
if let x = 5 {
    println!("{}", x);
};
```

- In `match` statements, all arms use refutable pattern except the last one that uses `_`, which uses irrefutable pattern.

#### Pattern Syntax

##### Inside the `match` expression

- Each `match` expression creates a new scope, hence varaibles defined in scope will shadow the variables defined outside.

  ```rust
      let x = Some(5);
      let y = 10;

      match x {
          Some(50) => println!("Got 50"),
          Some(y) => println!("Matched, y = {:?}", y), // This y will shadow the y defined outside of this scope
          _ => println!("Default case, x = {:?}", x),
      }

      println!("at the end: x = {:?}, y = {:?}", x, y);

      // Output =>
      // Matched, y = 5
      // at the end: x = Some(5), y = 10
  ```

- It's possible to use the "or" using the `|` syntax

  ```rust
      let x = 1;

      match x {
          1 | 2 => println!("one or two"),
          3 => println!("three"),
          _ => println!("anything"),
      }

      // Output =>
      // one or two
  ```

- You may also specify ranges in the arm, only numbers and `char` values are allowed.

  ```rust
      let x = 5;

      match x {
          1..=5 => println!("one through five"),
          _ => println!("something else"),
      }

      let y = 'c';

      match y {
          'a'..='j' => println!("early ASCII letter"),
          'k'..='z' => println!("late ASCII letter"),
          _ => println!("something else"),
      }

      // Output =>
      // one through five
      // early ASCII letter
  ```

##### Destructuring

- Destructuring structs

  ```rust
  struct Point {
      x: i32,
      y: i32,
  }

  fn main() {
      let p = Point { x: 0, y: 7 };

      // It's possible to destructure a struct into variables
      let Point { x: a, y: b } = p;
      assert_eq!(0, a);
      assert_eq!(7, b);

      // A shorthand, will directly store respective values in x and y
      let Point { x, y } = p;
      assert_eq!(0, x);
      assert_eq!(7, y);
  }
  ```

- Destructring as well as matching the structs

  ```rust
  fn main() {
      let p = Point { x: 0, y: 7 };

      match p {
          Point { x, y: 0 } => println!("On the x axis at {}", x),
          Point { x: 0, y } => println!("On the y axis at {}", y),
          Point { x, y } => println!("On neither axis: ({}, {})", x, y),
      }
  }
  ```

- Destructuring enums

  ```rust
  enum Color {
      Rgb(i32, i32, i32),
      Hsv(i32, i32, i32),
  }

  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(Color),
  }

  fn main() {
      let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

      match msg {
          Message::Quit => {
              println!("The Quit variant has no data to destructure.")
          }
          Message::Move { x, y } => {
              println!(
                  "Move in the x direction {} and in the y direction {}",
                  x, y
              );
          }
          Message::Write(text) => println!("Text message: {}", text),
          Message::ChangeColor(r, g, b) => println!(
              "Change the color to red {}, green {}, and blue {}",
              r, g, b
          ),
          Message::ChangeColor(Color::Rgb(r, g, b)) => println!(
              "Change the color to red {}, green {}, and blue {}",
              r, g, b
          ),
          Message::ChangeColor(Color::Hsv(h, s, v)) => println!(
              "Change the color to hue {}, saturation {}, and value {}",
              h, s, v
          ),
      }
  }
  ```

- We can mix, match, and nest destructuring patterns in even more complex ways

  ```rust
      let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
  ```

##### Ignoring Values

- You may use `_` or `..` to ignore values:

  - `_` : It is used when you want to ignore a warning of unused variable, inside match expression for the remaining values or using a name that starts with underscore.
  - `..` : Ignore remaining parts of the value.

- You may use `_` for ignoring an unused variable

  ```rust
  fn main() {
      let _x = 5;
      let y = 10;
  }
  ```

- It's possible to use `_` in functions

  ```rust
  fn foo(_: i32, y: i32) {
      println!("This code only uses the y parameter: {}", y);
  }

  fn main() {
      foo(3, 4);
  }
  ```

- There is a subtle difference between using only `_` and using a name that starts with an underscore.

  - The syntax `_x` still binds the value to the variable.

    ```rust
    // FAIL: s lost it's ownership to _s, but was attempted to use again for printing
    let s = Some(String::from("Hello!"));

    // _s binds the value, the value of s is moved
    if let Some(_s) = s {
        println!("found a string");
    }

    println!("{:?}", s);
    ```

  - Whereas `_` doesn’t bind at all.

    ```rust
        let s = Some(String::from("Hello!"));

        // _ never binds the value, hence s stays the owner
        if let Some(_) = s {
            println!("found a string");
        }

        println!("{:?}", s);
    ```

Note: Ignoring a function parameter can be especially useful in some cases, for example, when implementing a trait when you need a certain type signature but the function body in your implementation doesn’t need one of the parameters. The compiler will then not warn about unused function parameters, as it would if you used a name instead.

- Ignoring remaining parts of `struct` with `..`

  ```rust
  struct Point {
      x: i32,
      y: i32,
      z: i32,
  }

  let origin = Point { x: 0, y: 0, z: 0 };

  match origin {
      // It prevents using _ multiple times
      Point { x, .. } => println!("x is {}", x),
  }
  ```

- Skipping middle values using `..`

  ```rust
  fn main() {
      let numbers = (2, 4, 8, 16, 32);

      match numbers {
          (first, .., last) => {
              println!("Some numbers: {}, {}", first, last);
          }
      }
  }
  ```

- You can only use `..` once per tuple

  ```rust
  fn main() {
      let numbers = (2, 4, 8, 16, 32);

      match numbers {
          (.., second, ..) => {
              println!("Some numbers: {}", second)
          },
      }
  }
  ```

##### Match Guard

- _Match Guard_ is an additional `if` condition specified after the pattern in a match arm that must also match along with the pattern matching, _for that arm to be chosen_.

  ```rust
      let num = Some(4);

      match num {
          Some(x) if x % 2 == 0 => println!("The number {} is even", x),
          Some(x) => println!("The number {} is odd", x),
          None => (),
      }
  ```

- The downside of this additional expressiveness is that the compiler doesn't try to check for exhaustiveness when match guard expressions are involved.
- Using Match Guard with `|` operator

  ```rust
      let x = 4;
      let y = false;

      match x {
          // It works like this => (4 | 5 | 6) if y => ...
          // And not like this => 4 | 5 | (6 if y) => ...
          4 | 5 | 6 if y => println!("yes"),
          _ => println!("no"),
      }

      // Output =>
      // no
  ```

- `@` Bindings

  ```rust
  // The at operator (@) lets us create a variable that holds a value at the same time
  // we’re testing that value to see whether it matches a pattern.

  enum Message {
      Hello { id: i32 },
  }

  let msg = Message::Hello { id: 5 };

  match msg {
      Message::Hello {
          id: id_variable @ 3..=7,
      } => println!("Found an id in range: {}", id_variable), // Value of "id" is stored in "id_variable", hence it was knwon here
      Message::Hello { id: 10..=12 } => {
          println!("Found an id in another range") // Range was specified, Value of "id" is not stored inside any variable, hence it is unknown here
      }
      Message::Hello { id } => println!("Found some other id: {}", id), // Range was not specified, Value of "id" is stored inside "id", hence it was known here
  }
  ```
