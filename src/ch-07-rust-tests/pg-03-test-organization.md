# Test Organization

### Test Types

- The Rust community thinks about tests in terms of two main categories **Unit Tests** and **Integration tests**.

| Unit Tests                                                     | Integration tests                                               |
| -------------------------------------------------------------- | --------------------------------------------------------------- |
| Small and Focused                                              | Large and Broad                                                 |
| Internal                                                       | External                                                        |
| Tests One Module                                               | Tests Multiple Modules                                          |
| Can Test Private Interfaces                                    | Only Tests Public Interfaces                                    |
| Testing Internally such that external code may not possibly do | Testing like some external code would do                        |
| Lives inside `src` directory inside each module                | Lives in `tests` directory right outside `src` directory        |
| Module named tests inside each module with `#[cfg(test)]`      | Different files inside `tests` directory without `#[cfg(test)]` |

#### Unit Tests

- This annotation `#[cfg(test)]` tells Rust to only run this module on `cargo test`, not when you run `cargo build`.
- Thus, the functions following this annotation are never part of compiled result, hereby saving some space.
- Only used for unit tests, since integration tests are different directory they don't need to use it.
- An example worth noting:

  ```rust
  // Both code and unit test lives in the same file.

  pub fn public_fn() {
    ...
  }

  fn private_fn() {
    ...
  }

  // cfg stands for configuration
  #[cfg(test)]
  mod tests {
    // You'll use this line to pull code from outside this module but inside this file.
    use super::*;

    // A helper function
    fn helper() {
      ...
    }

    // Only this fn will be considered test, unlike above fn
    #[test]
    fn it_works() {
        // Can test both private and public functions.
        ...
    }
  }
  ```

#### Integration tests

- For integration test, we create a new directory `target` beside the `src` directory.
- Each file in the tests directory is compiled as its own separate crate.
- Treating each integration test file as its own crate is useful to create separate scopes that are more like the way end users will be using your crate.
- However, this means files in the tests directory don’t share the same behavior as files in src do.
- The file structure of integration tests are:

  ```rust
  rust_project
  ├── src
  |  └── lib.rs
  ├── target
  |  ├── ...
  |  └── ...
  ├── tests
  |  ├── common
  |  |  └── mod.rs // contains helper functions for tests
  |  └── integration_test.rs // contains integration tests
  ├── Cargo.lock
  └── Cargo.toml
  ```

- The helper functions lives inside file `tests/common/mod.rs`.
- This is a naming convention that rust uses to prevent functions inside this file not to appear in output logs of tests.
- Files in subdirectories of the tests directory don’t get compiled as separate crates or have sections in the test output.
- It looks something like this:

  ```rust
  pub fn setup() {
      // setup code specific to your library's tests would go here
  }
  ```

- The `integration_test.rs` looks similar to this:

  ```rust
  // Each file in the tests directory is a separate crate, so we need to bring our library into each test crate’s scope
  use adder;

  // Bring the common functions
  mod common;

  // No need to add `#[cfg(test)]` attribute, since we are in the tests directory.

  #[test]
  fn it_adds_two() {
      common::setup();
      assert_eq!(4, adder::add_two(2));
  }
  ```

- To run all the tests in a particular integration test file, use:

  ```zsh
  cargo test --test <integration-test-filename>
  ```

- We can't write integration tests for binary crates, the rust projects that only contains a `src/main.rs` file and doesn’t have a `src/lib.rs` file.
- The reason is that we cannot bring functions defined in the `src/main.rs` file into scope of files in `tests` directory with a use statement.
- Only library crates expose functions that other crates can use; binary crates are meant to be run on their own.
- Though, if a project contains both `src/lib.rs` and `sr/main.rs`, we can write integration tests for the important functionality inside `src/lib.rs` using the `use` keyword.
- If the important functionality works, the small amount of code in the `src/main.rs` file will work as well, and that small amount of code doesn’t need to be tested.

### Doc Tests

- You can write doc tests above the item, using the `Examples` with the docs comment `///` like this:

  ````rust
  /// Adds one to the number given.
  ///
  /// # Examples
  ///
  /// ```
  /// let arg = 5;
  /// let answer = my_crate::add_one(arg);
  ///
  /// assert_eq!(6, answer);
  /// ```
  pub fn add_one(x: i32) -> i32 {
      x + 1
  }
  ````

- Running `cargo test` will run the code examples in your documentation as tests! In case we change the function, the test will panic, and we'll require to update the docs to make it work.

  ```zsh
     Doc-tests my_crate

  running 1 test
  test src/lib.rs - add_one (line 5) ... ok

  test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
  ```

### Tests Output Log

- The output logs section has three parts:
  1. Unit Tests
  2. Integration Tests
  3. Doc Tests

