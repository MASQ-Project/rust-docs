# Useful Macros

### `todo!()`

- Indicates unfinished code.
- This can be useful if you are prototyping and are just looking to have your code typecheck.

    ```rust
    fn untested_function() {
        // let's not worry about implementing untested_function() for now
        todo!();
    }
    ```


### `dbg!()`

- This is a debug macro, we can pass any variable inside it to see debug logs in console.

  ```rust
  let my_variable = 12.0;
  dbg!(my_variable)
  ```
