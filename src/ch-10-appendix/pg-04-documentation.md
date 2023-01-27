## Documentation

- Always remember that, _programmers are interested in knowing how to use your crate as opposed to how your crate is implemented_.
- You'll be using `///` for documentation. Notice that you'll need to add docs comment above your item, not inside the `{}`. Using this`///`, we're documenting the item that follows it.

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

- You can use `cargo doc` to generate the docs as HTML, and can open it through `target/doc` directory or by running the command `cargo doc --open`.

- Just like `Examples` as shown in the above code, you can use other sections too. Here are other [commonly used sections](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commonly-used-sections).

- Running `cargo test` will run the code examples in your documentation as tests! In case we change the function, the test will panic, and we'll require to update the docs to make it work.

  ```zsh
     Doc-tests my_crate

  running 1 test
  test src/lib.rs - add_one (line 5) ... ok

  test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
  ```

- To document a whole crate, `//!` is used in the crate root file (`src/lib.rs`). Using this `//!`, we’re documenting the item that contains this comment.

  ```rust
  //! # My Crate
  //!
  //! `my_crate` is a collection of utilities to make performing certain
  //! calculations more convenient.
  ```
