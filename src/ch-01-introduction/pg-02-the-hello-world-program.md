# The `"Hello, World!"` program

Fun Fact: Rust is an ahead-of-time compiled language, meaning you can compile a program and give the executable to someone else, and they can run it even without having Rust installed.

- After [installing Rust](#installation), you may follow the following steps.

- Create a project folder, cd into it and create `main.rs` file:

  ```zsh
  mkdir hello_world
  cd hello_world
  touch main.rs
  ```

- Add the following program inside of it:

  ```rust
  // Filename: main.rs
  fn main() {
      println!("Hello, world!");
  }
  ```

- Some facts regarding the above code.

  - Main function is the first function that gets called.
  - `println!()` is not a function but a _macro_.
  - Macros contain an `!` mark.

- Compile and run the file:

  ```zsh
  rustc main.rs
  ```

  - For Linux and macOS:

    ```zsh
    ./main
    ```

  - For Windows:

    ```zsh
    .\main
    ```

- Alternatively, you may use the package manager [Cargo](#cargo) to create new boilerplate projects.