# Getting Started

- You can create a boilerplate rust project that automatically has tests using the following commmand:

  ```zsh
  cargo new <project-name> --lib
  ```

- A `test` in Rust is a function that’s annotated with the test attribute.
- The bodies of test functions typically perform these three actions:

  1. Set up any needed data or state.
  2. Run the code you want to test.
  3. Assert the results are what you expect.

- Two attributes to keep in mind:

  1. `#[test]` - To change a function into a test function, add `#[test]` on the line before `fn`.
  2. `#[should_panic]` - To declare before each test function that if this function panics then it is working correctly.

- After adding the attribute `#[test]`, the rust compiler is ready to run `cargo test`.

- When you run the command, behind the scenes Rust builds a test runner binary that runs the functions annotated with the `test` attribute.

- You can write the functions that are not tests inside the tests module, for example a helper function. So, the only way for the Rust to know whether a function is a test function is through the `#[test]` attribute.
- The tests fail when something in the test function panics.

- Here is the table for the logging statistics:

  | Statistic    | Meaning                                                                      |
  | ------------ | ---------------------------------------------------------------------------- |
  | Passed       | Passing Tests                                                                |
  | Failed       | Failing Tests                                                                |
  | Ignored      | Tests that were ignored due to `#[ignore]` attribute.                        |
  | Measured     | This is for benchmark tests that measure performance. (only in nightly Rust) |
  | Filtered Out | While running specific tests, the left out tests are called `filtered`.      |

- Here is the table for the macros you mauy use for assertion:

  | Assertion Macro | Use Case                                          | Argument(s)            |
  | --------------- | ------------------------------------------------- | ---------------------- |
  | `assert!()`     | If the condition is true then passes else panics. | Condition              |
  | `panic!()`      | Panics or fails the test with a message if given. | Message                |
  | `assert_eq!()`  | Passes if equal else panics. (`==`)               | (actual, expected)     |
  | `assert_ne!()`  | Passes if not equal else panics. (`!=`)           | (actual, not_expected) |

- In rust the convention doesn't matter, we can either use actual as first argument or as second. It is the programmer's convention.

- In case we are writing the tests in a module inside the same file then we'll need to use the `super::*;` inside the tests module to pull all the outside code of the current file.

```rust
//Filename: src/lib.rs
fn do_something() {
  ...
}


#[cfg(test)]
mod tests {
    // The line below will pull all the code of outer module inside.
    use super::*;

    #[test]
    fn test_do_something() {
      ...
    }
}
```

- For structs and enums that you define, you’ll need to implement `PartialEq` to assert that values of those types are equal or not equal.

  ```rust
  #[derive(PartialEq, Debug)]
  struct Rectangle {
      width: u32,
      height: u32,
  }

  #[cfg(test)]
  mod tests {
      use super::*;

      // This test will only work if we'll add #[derive(PartialEq)] to the struct or enum.
      #[test]
      fn rectangle_is_of_same_size() {
          let rectangle1 = Rectangle {
              width: 8,
              height: 7,
          };
          let rectangle2 = Rectangle {
              width: 8,
              height: 7,
          };

          assert_eq!(rectangle1, rectangle2);
      }
  }
  ```

- You’ll need to implement `Debug` to the struct or enum if you want to see the logs that say `(left != right)`.

  ```zsh
  ---- tests::rectangle_is_of_same_size stdout ----
  thread 'tests::rectangle_is_of_same_size' panicked at 'assertion failed: `(left == right)`
    left: `Rectangle { width: 8, height: 7 }`,
   right: `Rectangle { width: 8, height: 8 }`', src/lib.rs:56:9
  ```

- The `assert!()` macro also allows the message to show in case the test fails.

  ```rust
  assert!(
    result.contains("something"),
    "The result doesn't contain something. This was the actual result: {}",
    result
  )
  ```

- There is an attribute named `#[should_panic]`, that you can write before any test function to declare that if this function panics then it is working correctly.

  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;

      #[test]
      #[should_panic]
      fn greater_than_100() {
          Guess::new(200);
      }
  }
  ```

- This is the note that appears in case the function doesn't panics.

  ```zsh
  note: test did not panic as expected
  ```

- To make the `![should_panic]` attribute more precise we can add the `expected` parameter and pass a string to it such that the string is a substring of the relevant panic message.

  ```rust
  impl Guess {
      pub fn new(value: i32) -> Guess {
          if value < 1 {
              panic!(
                  "Guess value must be greater than or equal to 1, got {}.",
                  value
              );
          } else if value > 100 {
              panic!(
                  "Guess value must be less than or equal to 100, got {}.",
                  value
              );
          }

          Guess { value }
      }
  }

  #[cfg(test)]
  mod tests {
      use super::*;

      // The below test only passes if both the two conditions satisfies:
      // 1. Code should panic
      // 2. The string passed in expected parameter is a substring of the panic message.
      #[test]
      #[should_panic(expected = "Guess value must be less than or equal to 100")]
      fn greater_than_100() {
          Guess::new(200);
      }
  }
  ```

- There is also an alternative approach possible to use `Ok()` and `Err()` inside a test.

  ```rust
  #[cfg(test)]
  mod tests {
      #[test]
      fn it_works() -> Result<(), String> {
          if 2 + 2 == 4 {
              Ok(())
          } else {
              Err(String::from("two plus two does not equal four"))
          }
      }
  }
  ```

- Pros: The only upside of writing tests such that they return a `Result<T, E>` enables you to use the question mark (`?`) operator in the body of tests, which can be a convenient way to write tests that should fail if any operation within them returns an Err variant.

- Cons: If we write tests in above manner than we cannot use `#[should_panic]` attribute because we can use `Err()`.

