## Loops

- Rust has three kinds of loops:

  1. `loop` - Infinite Loop, uses `break` and `continue`
  2. `while` - Breaks when the condition doesn't meet.
  3. `for` - Faster and easier to use for iterators and classical for loops.

- Simple Infinite Loop:

  ```rust
  loop {
    // Do something iteratively
  }
  ```

- Named Loop

  ```rust
  'outer:loop {
   loop {
    break 'outer;
   }
  }
  ```

- Named Loop with different breaks:

  ```rust
  'oulter_loop: loop {
    loop {
      if condition {
        break 'oulter_loop; // Breaks Outer Loop
      }

      if some_other_condition {
        break; // Breaks Inner Loop
      }
    }
  }
  ```

- Returning values in loops:

  ```rust
  fn main() {
      let mut counter = 0;

      let result = loop {
          counter += 1;

          if counter == 10 {
              break counter * 2;
          }
      };

      println!("The result is {}", result);
  }
  ```

- The `while` loop:

  ```rust
  fn main() {
      let mut number = 3;

      // Prevents the use of break, by including the condition with while
      while number != 0 {
          println!("{}!", number);

          number -= 1;
      }

      println!("LIFTOFF!!!");
  }
  ```

- The `for` loop:

  ```rust
  // Last item in exclusive, or 0..10 === 0..=9
  for x in 0..10 {
      println!("{}", x); // x: i32
  }
  ```

- The `for` loop for iterator:

  ```rust
  fn main() {
      let a = [10, 20, 30, 40, 50];

      for element in a {
          println!("the value is: {}", element);
      }
  }
  ```

- A for loop for iterating characters in String

  ```rust
  for c in name.chars() {
      // c variable stores one charater per iteration
  }
  ```

- Enumeration

  ```rust
  for (i, v) in request.chars().enumerate() {
      // i is index, v is variable
  }
  ```

- The `for` loop in reverse:

  ```rust
  fn main() {
      for number in (1..4).rev() {
          println!("{}!", number);
      }
      println!("LIFTOFF!!!");
  }

  // Output:
  // 3!
  // 2!
  // 1!
  // LIFTOFF!!!
  ```
