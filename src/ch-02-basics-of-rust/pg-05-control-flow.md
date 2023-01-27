## Control Flow

### `if` expressions

- The code inside an `if` block is called an arm, similar to match.

  ```rust
  fn main() {
      let number = 3;

      if number < 5 {
          println!("condition was true"); // An arm
      } else {
          println!("condition was false");
      }
  }
  ```

- You can only pass a `bool` to the `if` expression

  ```rust
  // FAIL: number is of type integer and not bool
  fn main() {
      let number = 3;

      if number {
          println!("number was three");
      }
  }

  // This works since it's a condition
  fn main() {
      let number = 3;

      if number != 0 {
          println!("number was something other than zero");
      }
  }
  ```

- Rust only executes the block for the first true condition, and once it finds one, it doesn't even check the rest:

  ```rust
  fn main() {
      let number = 6;

      if number % 4 == 0 {
          println!("number is divisible by 4");
      } else if number % 3 == 0 {
          println!("number is divisible by 3"); // Only this statement will run
      } else if number % 2 == 0 {
          println!("number is divisible by 2");
      } else {
          println!("number is not divisible by 4, 3, or 2");
      }
  }
  ```

- Conditionals in Single Line:

  ```rust
  // This Works
  fn main() {
      let condition = true;
      let number = if condition { 5 } else { 6 };

      println!("The value of number is: {}", number);
  }

  // FAIL: Different data types integer and string
  fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {}", number);
  }
  ```
