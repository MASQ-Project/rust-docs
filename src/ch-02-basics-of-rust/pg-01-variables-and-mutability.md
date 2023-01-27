## Variables and Mutability

### Variables

- By default variables are immutable in Rust.
- Its advantages includes memory safety and easy concurrency.

- Example of immutable variable:

  ```rust
  let x = 5;
  ```

- Example of mutable variable:

  ```rust
  let mut x = 5;
  x = 6;
  ```

### Constants

- First, you aren’t allowed to use mut with constants.
- Constants aren’t just immutable by default—they’re always immutable.
- Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.
- Constants may be set only to a constant expression, not the result of a value that could only be computed at runtime.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

- Rust’s naming convention for constants is to use all uppercase with underscores between words.
- See the [Rust Reference’s section on constant evaluation](https://doc.rust-lang.org/reference/const_eval.html) for more information on what operations can be used when declaring constants.

### Shadowing

- You can declare a new variable with the same name as a previous variable, it's called _shadowing_.

  ```rust
  fn main() {
      let x = 5; // Binding x to value 5

      let x = x + 1; // Declaring a variable named x again, thereby performing shadowing

      {
          let x = x * 2; // Shadowing x again, but within the scope
          println!("The value of x in the inner scope is: {}", x);
      }

      println!("The value of x is: {}", x);
  }

  // Output:
  // The value of x in the inner scope is: 12
  // The value of x is: 6
  ```

- In shadowing, we can make a few transformations on a value but have the variable be immutable, unlike `let mut`.

- You can perform type conversions and still keep the same name.

  ```rust
  // Shadowing compiles without errors
  let spaces = "   ";
  let spaces = spaces.len();

  // It's not possible to change type of a mutable Variables
  let mut spaces = "   ";
  spaces = spaces.len(); // This line won't compile
  ```

### Shadowing Vs Mutable Variables

| Shadowing                                                              | Mutable Variables                                           |
| ---------------------------------------------------------------------- | ----------------------------------------------------------- |
| Transform a variable but still keep it as immutable.                   | We only make transformations after making variable mutable. |
| We need to declare variable with `let` everytime we perform shadowing. | We need to declare variable with only `let mut` once.       |
| It is possible to change the type of variable and keep the same name.  | It is not possible to change the type of mutable variable.  |
