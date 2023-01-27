### Project Structuring

- The Rust's _module system_ includes:
  - **Packages:** A Cargo feature that lets you build, test, and share crates
  - **Crates:** A tree of modules that produces a library or executable
  - **Modules** and **use:** Let you control the organization, scope, and privacy of paths
  - **Paths:** A way of naming an item, such as a struct, function, or module
- Once you’ve implemented an operation, other code can call that code via the code’s public interface without knowing how the implementation works.
- The way you write code defines which parts are public for other code to use and which parts are private.
  - `private` - No exteranl code can call this code directly
  - `public` - External code can call this code directly
- The way privacy works in Rust is that all items (functions, methods, structs, enums, modules, and constants) are private by default.

#### Package

- When we run the command `cargo new` it creates the package.
- A package contains a _Cargo.toml_ file that describes how to build those crates.
- A package can contain **at most one** _library_ crate. It can contain **as many** _binary_ crates as you’d like, but it must contain at least one crate (either library or binary).
- As a package grows, you can extract parts into separate crates that become external dependencies.

#### Crates

- A crate is a binary or library.
- The _crate root_ is a source file that the Rust compiler starts from and makes up the root module of your crate.
- Cargo follows a convention that _src/main.rs_ is the crate root of a **binary** crate with the same name as the package.
- Similarly, _src/lib.rs_ is the crate root of a **library** crate with the same name as the package.
- Cargo passes the crate root files to rustc to build the library or binary.
- A crate’s functionality is namespaced in its own scope, it means we can import another crate let's say `rand` which has a trait named `Rng`, and still create a new struct named `Rng` in our project's crate. The `rustc` will never confuse between the two and we can access the `rand`'s components as `rand::Rng`.

#### Modules

- Modules are used to structure code inside a crate.
- It is also used to provide privacy to your code.
  - `private` - Exteranl code outside that module can not call this code directly
  - `public` - External code outside that module can call this code directly
- By using modules, we can group related definitions together and name why they’re related.
- The benefit it'll provide you is that other programmers reading your code can easily find the code they are searching for because then they'll navigate through groups rather than each function definition. Also, they'll add new code in the right module.
- The contents of the files _src/main.rs_ and _src/lib.rs_ (these files are also referred as crate roots) form a module named `crate` at the root of the crate’s module structure, known as the _module tree_.

  ```zsh
  crate                                 // An implicit module, definitely not named by you
   └── front_of_house                   // Main Module inside lib.rs
       ├── hosting                      // Submodule
       │   ├── add_to_waitlist
       │   └── seat_at_table
       └── serving                      // Submodule
           ├── take_order
           ├── serve_order
           └── take_payment
  ```

#### Paths

- We use a path in the same way we use a path when navigating a filesystem.
- A path can take two forms:

  - An _absolute path_ starts from a crate root by using a crate name or a literal `crate`.
  - A _relative path_ starts from the current module and uses `self`, `super`, or an identifier in the current module.

- You ma consider paths in rust quite similar to the paths used to access the filesystem
  - `crate` - Root (`/`)
  - `::` - Used to distinct others (`/`)
  - `super` - Used to go back one step (`../`)
- Here's an exmaple:

  ```rust
  // eat_at_restaurant is a sibling to front_of_house (since they are in same file),
  // thus front_of_house doesn't need pub keyword to make it accessible.
  mod front_of_house {
      // Add pub to allow the functions that can access front_of_house to access hosting too.
      pub mod hosting {
          // Add pub to allow the functions that can access hosting to access add_to_waitlist too.
          pub fn add_to_waitlist() {}
      }
  }

  pub fn eat_at_restaurant() {
      // Absolute path
      // Filesystem Equivalent to /front_of_house/hosting/add_to_waitlist
      crate::front_of_house::hosting::add_to_waitlist();

      // Relative path
      // Filesystem Equivalent to front_of_house/hosting/add_to_waitlist
      front_of_house::hosting::add_to_waitlist();
  }
  ```

- Our preference is to specify absolute paths because it’s more likely to move code definitions and item calls independently of each other.
- Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules.
- But you can expose inner parts of child modules’ code to outer ancestor modules by using the pub keyword to make an item public.
- If you want to make an item like a function or struct private, you put it in a module.
- Another example to show usecase for `super`:

  ```rust
  fn serve_order() {}

  mod back_of_house {
      fn fix_incorrect_order() {
          cook_order();
          super::serve_order();
      }

      fn cook_order() {}
  }

  ```

- If we make a `struct` public, it **doesn't** mean all it's fields are public too. We use `.` to access fields.

  ```rust
  mod back_of_house {
      pub struct Breakfast {
          pub toast: String, // Accessible
          seasonal_fruit: String, // Not Accessible
      }

    ...
  }
  ```

- On the other hand, if we make an `enum` public all it's variants becomes public too. We use `::` to access variants.

  ```rust
  mod back_of_house {
      pub enum Appetizer {
          Soup, // Accessible
          Salad, // Accessible
      }

      ...
  }
  ```

#### The `use` keyword

- It is similar to the `import` keyword in python.

  ```rust
  mod front_of_house {
      pub mod hosting {
          pub fn add_to_waitlist() {}
      }
  }

  // The line below will make hosting as a valid name in the scope
  use crate::front_of_house::hosting;

  pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
  }
  ```

- We can use the following ways to achieve the same thing:
  - `use self::front_of_house::hosting;`
  - `use crate::front_of_house::hosting::add_to_waitlist;`
- Though, the one mentioned inside the code block is the idiiomatic way to do it in Rust.
- On the other hand, when bringing in structs, enums, and other items with use, it’s idiomatic to specify the full path.

  ```rust
  use std::collections::HashMap;

  fn main() {
      let mut map = HashMap::new();
      map.insert(1, 2);
  }
  ```

- In case if we have two items with same name (in our case `Result`) but from different crates (in our case `fmt` and `io`), then we'll not use the full path, as it'll confuse Rust.

  ```rust
  use std::fmt;
  use std::io;

  // This way Rust will be able to distinguish which Result we want
  fn function1() -> fmt::Result {
      // --snip--
  }

  fn function2() -> io::Result<()> {
      // --snip--
  }
  ```

- Alternatively, we can use the `as` keyword to deal with two same names.

  ```rust
  use std::fmt::Result;
  use std::io::Result as IoResult;
  ```

- We can re-export the code using `pub use`.

  ```rust
  mod front_of_house {
      pub mod hosting {
          pub fn add_to_waitlist() {}
      }
  }

  // The use keyword will create a local variable named hosting in this scope
  // and pub keyword will re-export it for the external code to use it.
  pub use crate::front_of_house::hosting;

  pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
  }
  ```

- Using external packages:

  - First, we'll add the name and version of the package in `cargo.toml`, so that it can be automatically downloaded through crates.io.

    ```toml
    rand = "0.8.3"
    ```

  - Then, we'll use the `use` keyword to bring it into the scope.

    ```rust
    use rand::Rng;

    fn main() {
        let secret_number = rand::thread_rng().gen_range(1..101);
    }
    ```

  - The packages like `std` is also external but is a part of Rust language and it is not needed to download it from crates.io.

- Nesting the paths:

  ```rust
  // Dirty Approach
  use std::cmp::Ordering;
  use std::io;

  // Cleaneer Aproach (Nested)
  use std::{cmp::Ordering, io};

  //Another Dierty Approach
  use std::io;
  use std::io::Write;

  // Cleaner Approach (Nesting using self)
  use std::io::{self, Write};
  ```

- The glob operator (`*`), is used to bring all public definitions into the scope.

  ```rust
  // It is a bit riskier, as it is hard to identify what all definitions have been brought into scope
  use std::collections::*;
  ```
