## Editions in Rust

- When you use, `cargo new`, Rust adds a bit of metadata to your `cargo.toml` about edition under `[package]`.

  ```toml
  edition = "2021"
  ```

- Here are the details about editions:

  | Edition | Description                                                     |
  | ------- | --------------------------------------------------------------- |
  | 2015    | If no edition is specified, your project is using this edition. |
  | 2018    | The Rust Book is written using this edition.                    |
  | 2021    | This is the latest release at the moment.                       |

- Rust has a 6-week release cycle.
- Rust releases small changes very often rather than big changes less often.
- Every 2-3 years, Rust team releases a new edition.
- Rust supports backward compatibility, it means even if you update your Rust software your old code will still compile.
- Here are the following cases you may consider:

  | Edition | Dependency |   Will Compile?    |
  | :-----: | :--------: | :----------------: |
  |  2015   |    2018    | :white_check_mark: |
  |  2018   |    2015    | :white_check_mark: |

- For more details, the [_Edition Guide_](https://doc.rust-lang.org/stable/edition-guide/) is a complete book about editions that enumerates the differences between editions and explains how to automatically upgrade your code to a new edition via cargo fix.
