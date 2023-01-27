# Modularizing

### Modularizing in different files

- Let's say we decided to put some code in the file `src/front_of_house.rs`, and we want to use this code inside the file `src/lib.rs`. It can be done like this:

  ```zsh
  src
   ├── front_of_house.rs
   └── lib.rs
  ```

  ```rust
  // Filename: src/lib.rs
  // This will bring the contents of module (thst is stored in file `src/front_of_house.rs`)
  // into the current file
  mod front_of_house;

  // This will allow us to use as well as export it
  // so that external can use it too.
  pub use crate::front_of_house::hosting;

  pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
  }
  ```

- Now, we can make a new directory as well and can store the files as folows:

  ```zsh
  src
   ├── front_of_house
   │   └── hosting.rs
   ├── front_of_house.rs
   └── lib.rs
  ```

  ```rust
  // Filename: src/front_of_house.rs
  pub mod hosting;
  ```

  ```rust
  // Filename: src/front_of_house/hosting.rs
  pub fn add_to_waitlist() {}
  ```

#### Re-exporting

- You may use re-exporting, for making it easier to use your crate for other developers. It allows to use the structures directly intead of following the heirarchy in which the crate is designed.

  - Without re-exporting:

    - How structure looks:

      ```rust
      pub mod kinds {
          pub enum PrimaryColor {
            ...
          }

          pub enum SecondaryColor {
            ...
          }
      }

      pub mod utils {
          use crate::kinds::*;

          pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
              ...
          }
      }
      ```

    - How others will be using it:

      ```rust
      use art::kinds::PrimaryColor;
      use art::utils::mix;

      fn main() {
          let red = PrimaryColor::Red;
          let yellow = PrimaryColor::Yellow;
          mix(red, yellow);
      }
      ```

  - With re-exporting:

    - How structure looks:

      ```rust
      // Here we're re-exporting it for direct use
      pub use self::kinds::PrimaryColor;
      pub use self::kinds::SecondaryColor;
      pub use self::utils::mix;

      pub mod kinds {
          ...
      }

      pub mod utils {
          ...
      }
      ```

    - How others will be using it:

      ```rust
      // Isn't it easier to import now?
      use art::mix;
      use art::PrimaryColor;

      fn main() {
        let red = PrimaryColor::Red;
        let yellow = PrimaryColor::Yellow;
        mix(red, yellow);
      }
      ```

#### Workspaces

- A `workspace` is a set of packages that share the same `Cargo.lock` and output directory.
- You can build your workspace that looks like this:

  ```zsh
  add
  ├── Cargo.lock
  ├── Cargo.toml
  ├── add_one
  │   ├── Cargo.toml
  │   └── src
  │       └── lib.rs
  ├── adder
  │   ├── Cargo.toml
  │   └── src
  │       └── main.rs
  └── target // Notice only one target directory
  ```

- The `cargo.toml` of `add` (the outer one) of the workspace will look like this:

  ```toml
  <!-- Filename: add/Cargo.toml -->
  [workspace]

  members = [
      "adder",
      "add_one",
  ]
  ```

- The workspace has one target directory at the top level, the `adder` package doesn’t have its own `target` directory.
- Even if we were to run `cargo build` from inside the `adder` directory, the compiled artifacts would still end up in `add/target` rather than `add/adder/target`.

- The `cargo.toml` of `adder` will look like this:

  ```toml
  <!-- Filename: add/adder/Cargo.toml -->
  [dependencies]
  add_one = { path = "../add_one" }
  ```

- The `main.rs` in `adder` will look something like this:

  ```rust
  // Filename: add/adder/src/main.rs
  use add_one;

  fn main() {
      let num = 10;
      println!(
          "Hello, world! {} plus one is {}!",
          num,
          add_one::add_one(num)
      );
  }
  ```

- To build the whole workspace, you may run this command from the `add` directory (the outer).

  ```zsh
  $ cargo build
     Compiling add_one v0.1.0 (file:///projects/add/add_one)
     Compiling adder v0.1.0 (file:///projects/add/adder)
      Finished dev [unoptimized + debuginfo] target(s) in 0.68s
  ```

- To run a particular package you may run the following command:

  ```zsh
  $ cargo run -p adder
      Finished dev [unoptimized + debuginfo] target(s) in 0.0s
       Running `target/debug/adder`
  Hello, world! 10 plus one is 11!
  ```

- All the dependencies in different packages will use the same version of the dependency. It is because the `cargo.toml` of the workspace will make only one entry of the dependency. It also saves space and makes all the package compatible with each other, since they'll be using the same version of the dependency.

- To run all test:

  ```zsh
  cargo run test
  ```

- To run test in particular file:

  ```zsh
  cargo test -p add_one
  ```

### Refactoring Guides

This pattern is about separating concerns: `main.rs` handles running the program, and `lib.rs` handles all the logic of the task at hand. Because you can’t test the main function directly, this structure lets you test all of your program’s logic by moving it into functions in `lib.rs`. The only code that remains in main.rs will be small enough to verify its correctness by reading it.