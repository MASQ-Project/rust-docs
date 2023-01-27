### Conventions in Rust

- Rust code uses _snake case_ as the conventional style for function and variable names, in which all letters are lowercase and underscores separate words.
- Rust style is to indent with four spaces, not a tab.
- Naming convention for constants is to use all uppercase with underscores between words.

  ```rust
  const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
  ```

- The names of static variables are in `SCREAMING_SNAKE_CASE` by convention.

- The names of a _type_ in rust uses CamelCase. For example, consider `T` in `Result<T>`.
- Documentation Comments use `///`, and is converted to HTML, unlike simple comments `//`. So simply add the documentation above the items. [Learn more](#documentation).
