### Installation

For Linux and macOS:

- Download the rustup and install it using:

  ```zsh
  curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
  ```

- You may need to install C compiler, because it'll give you a linker and also because some common Rust packages depend on C code:

  - For macOS:

    ```zsh
    xcode-select --install
    ```

  - For Linux:

    Linux users should generally install GCC or Clang, according to their distribution’s documentation. For example, if you use Ubuntu, you can install the `build-essential` package.

For Windows:

- People should follow [these instructions](https://www.rust-lang.org/tools/install) to install Rust. Also, install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/visual-cpp-build-tools/).
