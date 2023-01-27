## Functions

- We define a function in Rust by entering `fn` followed by a function name and a set of parentheses.
- The curly brackets tell the compiler where the function body begins and ends.
- The entry function to a rust's code is the `main` function.
- Rust doesn’t care where you define your functions, only that they’re defined somewhere.
- Here's an example of functions in rust:

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}

// Function with Parameter (or argument)
fn function_with_parameters(x: i32) {
    println!("The value of x is: {}", x);
}

// Function with two parameters
fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}

// Functions with a return value
// In rust, functions return last expression implicitly
fn five() -> i32 {
    5 // An Expression
}

// Functions returning through classical return keyword
// We use return keyword when we need to return early from a function
fn five() -> i32 {
    return 5; // A statement
}

// This will also work
fn plus_one(x: i32) -> i32 {
    x + 1
}

// Fail: Statement means, this function returns anything, expressed by (), a unit type
fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

### Statements and Expressions

- In Rust, function bodies are made up of a series of statements optionally ending in an expression.

#### Statements

- They are instructions that perform some action and do not return a value.
- They are just a standalone unit of execution.
- Creating a variable and assigning itself a value is a statement.

  ```rust
  let y = 6;
  ```

- Function definitions are also statements; the entire function is a statement in itself.

  ```rust
  fn main() {
      let y = 6;
  }
  ```

- Statements do not return values. Therefore, you can’t assign a let statement to another variable:

  ```rust
  fn main() {
      let x = (let y = 6); // FAIL : Statements doesn't return anything
  }
  ```

- In some languages, you can write `x = y = 6` and have both `x` and `y` have the value `6`; that is **not the case** in Rust.

#### Expressions

- They do not end with a semicolon, unlike statements.
- They evaluate into a resulting value.
- They are a combination of values and functions that are combined by the compiler to create a new value.
- The following things are considered as an expression:
  1. A simple math operation
  2. Calling a function
  3. Calling a macro
  4. A new scope block created with curly brackets
- A simple math operation is an expression:

  ```rust
  5 + 6
  ```

- In the below statement the standalone `6` is an expression.

  ```rust
  let x = 6; // A statement containing an expression
  ```

- A scope block is an expression:

  ```rust
  {
      let x = 3;
      x + 1
  }
  ```

- The above scope block will return `4` and can now become a part of a statement.

  ```rust
  fn main() {
      // The below statement contains the computed value of an expression
      let y = {
          let x = 3; // Statements end with a semicolon
          x + 1 // Expressions do not end with semicolon
      };

      println!("The value of y is: {}", y);
  }
  ```