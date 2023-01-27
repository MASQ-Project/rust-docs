## Functional Language Features

Fun Fact: The implementations of closures and iterators are such that runtime performance is not affected. It means as if you've written to an optimized low level code, like in Assembly Language. This is part of Rust’s goal to strive to provide _zero-cost abstractions_.

- Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, assigning them to variables for later execution, and so forth.
- Specifically, functional programming includes:
  - _Closures_: A function-like construct you can store in a variable.
  - _Iterators_: A way of processing a series of elements.

#### Closures

- They are "Anonymous Functions that Can Capture Their Environment".
- An example of closure:

  ```rust
  let expensive_closure = |num| {
      println!("calculating slowly...");
      thread::sleep(Duration::from_secs(2));
      num
  };
  ```

- Why closures don't require type annotations, but functions (`fn`) do?

  - Type annotations are required on functions because they’re part of an explicit interface exposed to your users. Defining this interface rigidly is important for ensuring that everyone agrees on what types of values a function uses and returns.
  - But closures aren’t used in an exposed interface like this: they’re stored in variables and used without naming them and exposing them to users of our library.

- In case, we still want to explicitly define type annotations, we can do it by:

  ```rust
  let expensive_closure = |num: u32| -> u32 {
      println!("calculating slowly...");
      thread::sleep(Duration::from_secs(2));
      num
  };
  ```

- Comparisons for Functions and closures syntax:

  ```rust
  fn  add_one_v1   (x: u32) -> u32 { x + 1 }
  let add_one_v2 = |x: u32| -> u32 { x + 1 };
  let add_one_v3 = |x|             { x + 1 };
  let add_one_v4 = |x|               x + 1  ;
  ```

- Closures will always have only one concrete type:

  ```rust
  // FAIL: Closure inferred two different types of x, which is against the rules
  let example_closure = |x| x;

  let s = example_closure(String::from("hello"));
  let n = example_closure(5);
  ```

- Performing _memoization_ or _lazy evaluation_:

  - We can create a struct that will hold the closure and the resulting value of calling the closure.
  - The struct will execute the closure only if we need the resulting value, and it will cache the resulting value so the rest of our code doesn’t have to be responsible for saving and reusing the result.
  - All closures implement at least one of the traits: `Fn`, `FnMut`, or `FnOnce`.
  - Using this information, we can create a `Cacher`

    ```rust
    // Cacher will store a closure inside calculation
    // and the computed value inside value
    struct Cacher<T>
    where
        T: Fn(u32) -> u32,
    {
        calculation: T,
        value: Option<u32>,
    }
    ```

  - The use of the memoization is that, we can store the closure, that contains computation which takes very long time to finish. Then we can save it's computed value inside the struct, so that we can reuse to computation (thereby preventing expensive computation again and again), as well as update the computed value whenever necessary.
  - We'll need to write an implementation block to make the `Cacher` easier to use:

    ```rust
    impl<T> Cacher<T>
    where
        T: Fn(u32) -> u32,
    {
        fn new(calculation: T) -> Cacher<T> {
            Cacher {
                calculation,
                value: None
            }
        }

        fn value(&mut self, arg: u32) -> u32 {
            match self.value {
                Some(v) => v,
                None => {
                    let v = (self.calculation)(arg);
                    self.value = v;
                    v
                }
            }
        }
    }
    ```

  - The only limitation of this `Cacher` is that it assumes that, it'll only receive one value, that means even if we call the `value()` function multiple with different arguments, it'll still return the same value every time and that value will be the computed value when the closure was called for the first time. Here's the explanation:

    ```rust
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1); // v1 = 1
    let v2 = c.value(2); // v2 = 1
    let v3 = c.value(3); // v3 = 1
    ```

  - So, here is a better version of the above cacher that can memorize all the arguments and their computed value inside a HashMap, which is also generic. You may refer it's implementation over [here](https://gist.github.com/utkarshg6/c8a5cb39ef89b8f16fcce3098754c001).

- Capturing the Environmet with closures:

  - You can't don the following thing using simple functions:

    ```rust
    // FAIL: Functions can't capture their environment, hence x shouldn't live inside the function
    fn main() {
        let x = 4;

        fn equal_to_x(z: i32) -> bool {
            z == x
        }

        let y = 4;

        assert!(equal_to_x(y));
    }
    ```

  - But, you can easliy do this using closure:

    ```rust
    fn main() {
        let x = 4;

        let equal_to_x = |z| z == x;

        let y = 4;

        assert!(equal_to_x(y));
    }
    ```

  - Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: taking ownership, borrowing mutably, and borrowing immutably. These are encoded in the three Fn traits as follows:

    - `FnOnce` consumes the variables it captures from its enclosing scope, known as the closure’s environment. To consume the captured variables, the closure must take ownership of these variables and move them into the closure when it is defined. The Once part of the name represents the fact that the closure can’t take ownership of the same variables more than once, so it can be called only once.
    - `FnMut` can change the environment because it mutably borrows values.
    - `Fn` borrows values from the environment immutably.

  - When you create a closure, Rust infers which trait to use based on how the closure uses the values from the environment. All closures implement `FnOnce` because they can all be called at least once. Closures that don’t move the captured variables also implement `FnMut`, and closures that don’t need mutable access to the captured variables also implement `Fn`. In Listing 13-12, the equal_to_x closure borrows x immutably (so equal_to_x has the Fn trait) because the body of the closure only needs to read the value in x.

  - If you want to force the closure to take ownership of the values it uses in the environment, you can use the `move` keyword before the parameter list. This technique is mostly useful when passing a closure to a new thread to move the data so it’s owned by the new thread. The `move` closures may still implement Fn or FnMut, even though they capture variables by move. This is because the traits implemented by a closure type are determined by what the closure does with captured values, not how it captures them. The move keyword only specifies the latter.

  - An example of `move`:

    ```rust
    // FAIL: We tried to print x even though it is moved inside closure
    fn main() {
        let x = vec![1, 2, 3];

        let equal_to_x = move |z| z == x;

        println!("can't use x here: {:?}", x);

        let y = vec![1, 2, 3];

        assert!(equal_to_x(y));
    }
    ```

  - Most of the time when specifying one of the `Fn` trait bounds, you can start with `Fn` and the compiler will tell you if you need `FnMut` or `FnOnce` based on what happens in the closure body.

#### Iterators

- Iterators are lazy in rust, meaning they have no effect until you call methods that consume the iterator to use it up.

  ```rust
  let v1 = vec![1, 2, 3];

  let v1_iter = v1.iter(); // It won't do anything useful until called upon

  for val in v1_iter { // Now, the iterator is called upon and used
      println!("Got: {}", val);
  }
  ```

- To just get the next element stored in iterator:

  ```rust
  let v1 = vec![1, 2, 3];

  // Calling the next() method changes the state of iterator,
  // hence we'll need to use mut in this case
  let mut v1_iter = v1.iter();

  assert_eq!(v1_iter.next(), Some(&1));
  assert_eq!(v1_iter.next(), Some(&2));
  assert_eq!(v1_iter.next(), Some(&3));
  assert_eq!(v1_iter.next(), None);
  ```

- Why is it required to use `mut` when using `next()`, but not when using `for` loop?

  - `next()` - Each call to `next` eats up an item from the iterator. Hence, we need to define it as `mut` to be able to do that.
  - `for` - The loop takes ownership of the iterator and made it mutable behind the scenes. Hence, we don't need to use `mut` there.

- Difference between `iter`, `into_iter`, and `iter_mut`

  - They all return iterator, except the way they return differs. Here are the differences:

    - `into_iter`: It yields any of `T`, `&T` or `&mut T`, depending on the context.
    - `iter`: It yields `&T`.
    - `iter_mut`: It yields `&mut T`.

  - For more details refer to this [stackoverflow answer](https://stackoverflow.com/questions/34733811/what-is-the-difference-between-iter-and-into-iter).

- `Consuming Adaptors`: Some methods inside `Iterator` trait uses `next()`. It means those functions will also eat away the iterator, just like how `next()` does. Here's an example:

  ```rust
  let v1 = vec![1, 2, 3];

  let v1_iter = v1.iter();

  let total: i32 = v1_iter.sum(); // sum() uses the next() and hence will eat away the iterator
  ```

- `Iterator Adaptors`: Some methods inside `Iterator` allows you to change iterators into different kinds of iterators. It is also possible to use `Iterator`, `Enumerator`, `Map`, and `Filter` together. Rust has these functions inside the Iterator Trait.

  ```rust
  let v1: Vec<u32> = vec![0, 1, 2, 3, 4, 5];
  let iterator = v1.iter()
                   .enumerate()
                   .filter(|(i, val)| i % 2 == 0)
                   .map(|(i, val)| val); // On it's own it won't do anything, because iterators are lazy

  // You can either print them one by one using for loop (remember it'll consume the iterator)
  for val in iterator {
      println!("{}", val);
  }

  // Or you can collect them inside a vector, make sure you explicitly specify the type (`Vec<_>`) too.
  let new_vector: Vec<_> = iterator.collect();
  println!("New Vector: {:?}")
  ```

- Creating your own iterator:

  - You'll need to implement `Iterator` trait on your type.
  - You'll only need to define one function, that is `next()`, it'll be sufficient.

    ```rust
    struct Counter {
        count: u32,
    }

    impl Counter {
        fn new() -> Self {
            Self {
                count: 0,
            }
        }
    }

    impl Iterator for Counter {
        type Item = u32;

        fn next(&mut self) -> Option<Self::Item> {
            if self.count < 5 {
                self.count += 1;
                Some(self.count)
            } else {
                None
            }
        }
    }

    fn main() {
        let counter = Counter::new();
        for val in counter {
            println!("{:?}", val);
        }

        // Since we have next() method we can use any default implementation of the Iterator trait
        let sum: u32 = Counter::new()
            .zip(Counter::new().skip(1)) // Skips first element only and generate pairs { (1,2) (2,3) (3,4) (4,5) } because (5,None) is ignored
            .map(|(a, b)| a * b) // [2, 6, 12, 20]
            .filter(|x| x % 3 == 0) // [6, 12]
            .sum(); // 18
    }
    ```

- Which is faster, `for` loop or `iterator adapters`?

  - Here are the benchmarks:

    ```zsh
    test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
    test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
    ```

  - Iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s `zero-cost abstractions`, which means that using the abstraction imposes no additional runtime overhead.
