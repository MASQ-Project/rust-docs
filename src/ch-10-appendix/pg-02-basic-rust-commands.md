# Basic Rust Commands

- To check version or to verify that Rust is installed properly:

  ```zsh
  rustc --version
  ```

- Update Rust:

  ```zsh
  rustup update
  ```

- Uninstall Rust and rustup:

  ```zsh
  rustup self uninstall
  ```

- Open Rust Docs locally on browser:

  ```zsh
  rustup doc
  ```

- List the rustup toolchain

  ```zsh
  rustup toolchain list
  ```

- Install rustup toolchain

  ```zsh
  rustup toolchain install nightly-x86_64-unknown-linux-gnu
  ```

- Use nightly on a specific project (after this all the `rustc` and `cargo` commands will use nightly):

  ```zsh
  cd ~/projects/needs-nightly
  rustup override set nightly
  ```
