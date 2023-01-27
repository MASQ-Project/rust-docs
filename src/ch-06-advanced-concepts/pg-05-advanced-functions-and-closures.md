## Advanced Functions and Closures

#### Pass functions to the functions

- Just like you can pass closures to the functions, you can also pass "funtions to the functions".
- There's a type represented by `fn`, (don't confuse it with `Fn`, that is a closure trait).
- This `fn` type is called _Function Pointer_.

- Here's how you can use it:

  ```rust
  fn add_one(x: i32) -> i32 {
    x + 1
  }

  fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
  }

  fn main() {
    let answer = do_twice(add_one, 5);
    println!("Answer: {}", answer);
  }
  ```

- Function pointers implement all three of the closure traits (`Fn`, `FnMut`, and `FnOnce`), so you can always pass a function pointer as an argument for a function that expects a closure.
- So, instead of passing a closure, you can simply enter a function name, and it will work.

  ```rust
  // You can either do this
  let list_of_numbers = vec![1, 2, 3];
  let list_of_strings: Vec<String> =
      list_of_numbers
      .iter()
      .map(|i| i.to_string()) // Provide the closure
      .collect();


  // Or you can simply enter the function name
  let list_of_numbers = vec![1, 2, 3];
  let list_of_strings: Vec<String> =
      list_of_numbers
      .iter()
      .map(ToString::to_string) // Enter the function name, wohoo!
      .collect();
  ```

- You can use enum variants as an initializer function. Also, we now know we can pass a function inside a function, so here's what we can also do:

  ```rust
  enum Status {
      Value(u32), // So, this works as an initializer function too
      Stop,
  }

  let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect(); // This will create Status instances of the variant Value
  ```

- You can do the same thing by using closures, it's more of a personal preference. You can use whichever feels more clearer to you.

#### Return Closures

- First of all, you can't return functions, that's not allowed in rust.
- Technically, you are not alllowed to use `fn` (Funciion Pointer) as a return type, but you can return closures.
- Closures are represented by traits.
- To return a type that implements a trait, you can do either of the two:
  - Return a concrete type
  - Use dynamic dispatch (it'll allow the function to know concrete type at runtime).
- Closures don't have the concrete type, you can't send them directly, you'll need to use dynamic dispatch.

  ```rust
  // FAIL: This will also not work
  fn returns_closure() -> dyn Fn(i32) -> i32 {
      |x| x + 1
  }
  ```

- Now, Rust needs one more thing, the size of the closure. Remember the `Sized` trait.
- The solution to this problem, is to wrap the return type with some sort of pointer, in case of strings we use references. For example, using `&str` instead of `str`.
- Here, we'll use `Box<T>` to simply store it on the heap.

  ```rust
  // This is how you can successfully return a closure
  fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
      Box::new(|x| x + 1)
  }
  ```

- In case you want to understand better how you can use traits with dynamic dispatch, you can check [here](#defining-a-common-behaviour-using-trait).
