## Cargo

- It is the package manager for rust. It does the following things:

  - Manages Rust Projects
  - Download packages or dependencies
  - Build both your code and it's dependencies

- Check Cargo Installation or Version:

  ```bash
  cargo --version
  ```

- Create New Boilerplate Project:

  ```bash
  cargo new {project-name}
  ```

- Create Boilerplate Binary Project (only `main.rs`):

  ```bash
  cargo new {project-name} --bin
  ```

- Create Boilerplate Library Project (for writing tests, contains `lib.rs`):

  ```bash
  cargo new {project-name} --lib
  ```

- Help for Cargo new:

  ```bash
  cargo new --help
  ```

- Build a Project (Installing Dependencies and Compiling):

  ```bash
  cargo build
  ```

- Run a Project (build + run):

  ```bash
  cargo run
  ```

- Running through executable binary:

  ```bash
  ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
  ```

- Compile but don't generate executable (it's just faster than `cargo build`):

  ```bash
  cargo check
  ```

- Build for releases (it's optimized and binaries lives in `target/release`):

  ```bash
  cargo build --release
  ```

- To generate docs, you can access them through `target/doc`:

  ```zsh
  cargo doc
  ```

- To generate docs of all dependencies of your project and run them in browser:

  ```zsh
  cargo doc --open
  ```

- We can install `cargo-expand` to use cargo libraries system wide.

  ```zsh
  cargo install cargo-expand
  ```

- The cargo expand command

  ```zsh
  cargo expand
  ```

- To install a package from crates.io into your system, you may use this command (only binary crates can be installed and ensure that `$HOME/.cargo/bin` is in your `$PATH`):

  ```zsh
  cargo install <package-name>
  ```

- To list out custom cargo commands:

  ```zsh
  cargo --list
  ```

- To automatically format code:

  ```zsh
  cargo fmt
  ```

- To automatically fix warnings (fixable by compiler):

  ```zsh
  cargo fix
  ```

- To Lint your code:

  ```zsh
  cargo clippy
  ```

#### The `opt-level`

- The `opt-level` setting controls the number of optimizations Rust will apply to your code, with a range of `0` to `3`.

  ```toml
  // Filename: Cargo.toml
  [profile.dev]
  opt-level = 0 // Less Optimization, less compiling time

  [profile.release]
  opt-level = 3 // More Optimizations, more compiling time
  ```

- You can override any default setting by adding a different value for it in `Cargo.toml`. To override, you can add these two lines below the above lines.

  ```toml
  // Filename: Cargo.toml
  [profile.dev]
  opt-level = 1
  ```

- To learn more about customizing profiles, you can read the docs [here](https://doc.rust-lang.org/cargo/reference/profiles.html).

For more information about Cargo, check out [its documentation](https://doc.rust-lang.org/cargo/).
