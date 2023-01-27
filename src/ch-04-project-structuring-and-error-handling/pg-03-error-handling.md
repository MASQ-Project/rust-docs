
## Error Handling

#### Types of Errors

| Recoverable                                                                | Unrecoverable                                                                |
| -------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Errors like file not found error.                                          | Errors like trying to access a location beyond the end of an array.          |
| It’s reasonable to report the problem to the user and retry the operation. | They are always symptoms of bugs.                                            |
| `Result<T, E>` is used for Recoverable Errors.                             | The `panic!` macro is used to stop the execution for an unrecoverable error. |

#### Unrecoverable Errors with `panic!`

Fun Fact: In C, attempting to read beyond the end of a data structure is undefined behavior. You might get whatever is at the location in memory that would correspond to that element in the data structure, even though the memory doesn’t belong to that structure. This is called a buffer overread and can lead to security vulnerabilities if an attacker is able to manipulate the index in such a way as to read data they shouldn’t be allowed to that is stored after the data structure. In Rust, you'll encounter a `panic!()` in such cases.

- When the `panic!` macro executes, Rust does the following:

  - Print a failure message
  - Unwind and clean up the stack
  - Quit

- Panic is usually used, when a bug appears and the programmer doesn't know how to handle it.
- If you don't want your program to "slowly unwind and clean up the stack" instead "abort the program and let OS handle the cleaning". You may do that by adding following lines to the `Cargo.toml` file. Refer [here](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic) for more.

  ```rust
  [profile.release]
  panic = 'abort'
  ```

- To receive a backtrace in case of panic, you might need to run the following command:

  ```zsh
  RUST_BACKTRACE=1 cargo run
  ```

- The best way to read backtraces is to ready from top to bottom, once you see the first instance mentioning a file that you've written, you should probably try to solve from there.

- Debug symbols (they are required to receive backtraces) are enabled by default when using `cargo build` or `cargo run` without the `--release` flag.

#### Recoverable Errors with `Result`

- Result is an enum, that considers two possible outcomes: success (`Ok(T)`) or failure (`Err(E)`).

  ```rust
  enum Result<T, E> {
      Ok(T),
      Err(E),
  }
  ```

- Handling recoverable errors using the `match` expression.

  ```rust
  use std::fs::File;

  fn main() {
      let f = File::open("hello.txt");

      let f = match f {
          Ok(file) => file, // Handling Success
          Err(error) => panic!("Problem opening the file: {:?}", error), // Handling Failure
      };
  }
  ```

- Matching on different errors:

  ```rust
  use std::fs::File;
  use std::io::ErrorKind;

  fn main() {
      let f = File::open("hello.txt");

      // Match on File, whether it gets opened or not
      let f = match f {
          Ok(file) => file,
          // If file not found, then create a new file and transfer file handle,
          // this error is part of io::ErrorKind, which was found using error.kind()
          Err(error) => match error.kind() {
              // In case we receive ErrorKind::NotFound, we'll apply
              // match again to check whether creation of file, fails or succeeds
              ErrorKind::NotFound => match File::create("hello.txt") {
                  Ok(fc) => fc,
                  Err(e) => panic!("Problem creating the file: {:?}", e),
              },
              other_error => {
                  panic!("Problem opening the file: {:?}", other_error)
              }
          },
      };
  }
  ```

- In case you don't like using a lot of match statements (refer above example), you may use `unwrap_or_else`:

  ```rust
  use std::fs::File;
  use std::io::ErrorKind;

  fn main() {
     let f = File::open("hello.txt").unwrap_or_else(|error| {
       if error.kind() == ErrorKind::NotFound {
         File::create("hello.txt").unwrap_or_else( |error| {
           panic!("Problem creating the file: {:?}", error);
         }
         )
       } else {
         panic!("Problem opening the file: {:?}", error);
       }
     })
  }
  ```

- In case you want a shortcut, you may only use `unwrap()`. It either returns what's inside `Ok(T)`, or panics in case of `Err(E)`:

  ```rust
  use std::fs::File;

  fn main() {
      let f = File::open("hello.txt").unwrap();
  }
  ```

- For those cases, when you want to send a panic message but only want to unwrap in one line, you may use `expect`:

  ```rust
  use std::fs::File;

  fn main() {
      let f = File::open("hello.txt").expect("Failed to open hello.txt"); // Same as unwrap but contains panic message
  }
  ```

- Propogating errors using the `Result`:

  ```rust
  use std::fs::File;
  use std::io::{self, Read};

  fn read_username_from_file() -> Result<String, io::Error> {
      let f = File::open("hello.txt");

      let mut f = match f {
          Ok(file) => file,
          Err(e) => return Err(e), // This is a std::io error type
      };

      let mut s = String::new();

      match f.read_to_string(&mut s) {
          Ok(_) => Ok(s),
          Err(e) => Err(e), // This is also a std::io error type
      }
  }
  ```

- The shortcut of above code can be done using `?`. `unwrap` panics in case of Err(E), but this operator returns the error, same as the code above.

  ```rust
  // ? operator changes the error type to the mentioned
  // Error type in the fn declaration using the from implementation
  use std::fs::File;
  use std::io;
  use std::io::Read;

  fn read_username_from_file() -> Result<String, io::Error> {
      let mut f = File::open("hello.txt")?;
      let mut s = String::new();
      f.read_to_string(&mut s)?;
      Ok(s)
  }
  ```

- It is possible to use the `?` operator multiple times in a single line:

  ```rust
  use std::fs::File;
  use std::io;
  use std::io::Read;

  fn read_username_from_file() -> Result<String, io::Error> {
      let mut s = String::new();

      File::open("hello.txt")?.read_to_string(&mut s)?;

      Ok(s)
  }
  ```

- There's a Rust's official implementation of the functionality mentioned in the above code:

  ```rust
  use std::fs;
  use std::io;

  fn read_username_from_file() -> Result<String, io::Error> {
      fs::read_to_string("hello.txt")
  }
  ```

- The `?` operator can only be used in the functions that has a return type of `Result<Ok(T), Err(E)>`, `Option<Some(T), None>`, or another type that implements FromResidual:

  ```rust
  // FAIL: main() doensn't returns a Result<>
  // but the ? operator requires that
  use std::fs::File;

  fn main() {
      let f = File::open("hello.txt")?;
  }
  ```

  ```rust
  // It works with the Option
  fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
  }
  ```

- There's a way to use `?` inside `main()`. The `main()` either returs `0` on success or other integer on failure. Also, it's possible to return `<Result(), E>`:

  ```rust
  use std::error::Error;
  use std::fs::File;

  fn main() -> Result<(), Box<dyn Error>> {
      let f = File::open("hello.txt")?;

      Ok(())
  }
  ```

- Differences between `unwrap`, `unwrap_or`, and `?` operator

| Property                                        | `unwrap`           | `expect`                      | `unwrap_or`                           | `?` operator       |
| ----------------------------------------------- | ------------------ | ----------------------------- | ------------------------------------- | ------------------ |
| Error Handling                                  | Panics             | Panics with the given message | Executes code inside it's parantheses | Returns error      |
| Can be used on `Result`                         | :heavy_check_mark: | :heavy_check_mark:            | :heavy_check_mark:                    | :heavy_check_mark: |
| Can be used on `Option`                         | :heavy_check_mark: | :heavy_check_mark:            | :heavy_check_mark:                    | :heavy_check_mark: |
| Function return type to be same as wrapped item | :x:                | :x:                           | :x:                                   | :heavy_check_mark: |

Note: You can only use the `?` operator on a `Result` in a function that returns `Result`, and you can use the `?` operator on an `Option` in a function that returns `Option`.

- To `panic!` or Not to `panic!`

  - When to use `Result`

    - When `panic!` is called, there is no way to recover the program, so if there is a slightest possiblity to recover the program, it's recommended to use that instead of `panic!`.
    - Always try to prevent converting a recoverable error into an unrecoverable one. Hence, always prefer `Result` over `panic!`.
    - The `unwrap` and `expect` methods are very handy when prototyping, and if you want to make your program more robust, you may add better error handling.

  - When to use `panic!`

    - In case you want your test to fail in certain cases, even if a certain fn is not exactly what the test is for, it's better to `panic!` in those situations.
    - It’s advisable to have your code panic when it’s possible that your code could end up in a `bad state`. The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format. You don't want to carry this bad state throughout the program and instead would prefer it to end through `panic!`.
    - If someone calls your code and passes in values that don’t make sense, the best choice might be to call `panic!` and alert the person using your library to the bug in their code so they can fix it during development.
    - Similarly, `panic!` is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.
    - When your code performs operations on values, your code should verify the values are valid first and panic if the values aren’t valid. This is mostly for safety reasons: attempting to operate on invalid data can expose your code to vulnerabilities.
    - However, having lots of error checks in all of your functions would be verbose and annoying. Fortunately, you can use Rust’s type system (and thus the type checking the compiler does) to do many of the checks for you. If your function has a particular type as a parameter, you can proceed with your code’s logic knowing that the compiler has already ensured you have a valid value. For example, if you have a type rather than an Option, your program expects to have something rather than nothing.
    - Another example is using an unsigned integer type such as u32, which ensures the parameter is never negative.

  - When to call `unwrap()`

    - In case you exactly know that the code won't `panic!`, then it's better to use `unwrap()`, and stop caring about the other possibilities. Here's an Example:

      ```rust
      use std::net::IpAddr;

      // Compile isn't smart enough to see this string is a valid IP address
      // but we are
      let home: IpAddr = "127.0.0.1".parse().unwrap();
      ```
