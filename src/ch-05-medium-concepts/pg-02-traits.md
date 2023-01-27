
### Traits

- A _trait_ tells the Rust compiler about functionality a particular type has and can share with other types.
- They are similar to the interfaces in other languages. They are used to define the method signature.
- You may define the code implementations inside the `impl` block of the types that implement that trait.
- It is also possible to define a default implementation and then override it in `impl` block.
- Use cases:

  - For example, you're creating a library that wants to summarize an article or a tweet. We want to implement this shared functionality.
  - We can define a trait to define the interface of this functionality.

    ```rust
    pub trait Summary {
        fn summarize(&self) -> String;
    }
    ```

  - Now each type implementing this trait must provide its own custom behavior for the body of the method.

#### Implementing a trait

- Implementing trait on different types

  ```rust
  pub struct NewsArticle {
      pub headline: String,
      pub location: String,
      pub author: String,
      pub content: String,
  }

  impl Summary for NewsArticle {
      fn summarize(&self) -> String {
          format!("{}, by {} ({})", self.headline, self.author, self.location)
      }
  }

  pub struct Tweet {
      pub username: String,
      pub content: String,
      pub reply: bool,
      pub retweet: bool,
  }

  impl Summary for Tweet {
      fn summarize(&self) -> String {
          format!("{}: {}", self.username, self.content)
      }
  }
  ```

- Using types that implements trait:

  ```rust
  // You'll require to pull both trait along with the desired type
  use aggregator::{Summary, Tweet};

  fn main() {
      let tweet = Tweet {
          username: String::from("horse_ebooks"),
          content: String::from(
              "of course, as you probably already know, people",
          ),
          reply: false,
          retweet: false,
      };

      println!("1 new tweet: {}", tweet.summarize());
  }
  ```

#### Restrictions

- One restriction to note with trait implementations is that we can implement a trait on a type only if at least one of the trait or the type is local to our crate.
  - For example, we can implement standard library traits like `Display` on a custom type like `Tweet` as part of our `aggregator` crate functionality, because the type Tweet is local to our aggregator crate.
  - But we can’t implement external traits on external types. For example, we can’t implement the `Display` trait on `Vec<T>` within our `aggregator` crate, because `Display` and `Vec<T>` are defined in the standard library and aren’t local to our `aggregator` crate.
- This restriction is part of a property of programs called _coherence_, and more specifically the _orphan rule_, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa.

#### Default Implementations

- We can define a default implementation by adding code inside the method signatures of traits.

  ```rust
  pub trait Summary {
      fn summarize(&self) -> String {
          String::from("(Read more...)")
      }
  }
  ```

- To use default implementation on a type, we can do that by using empty braces `{}`.

  ```rust
  impl Summary for NewsArticle {}
  ```

- It is possible to keep a trait with a mix of method signatures and method signatures with default implementations.

  ```rust
  pub trait Summary {
      fn summarize_author(&self) -> String;

      fn summarize(&self) -> String {
          format!("(Read more from {}...)", self.summarize_author())
      }
  }
  ```

- Now we only need to require to define `summarize_author` method inside the `impl` block of the type that's implementing the trait.

  ```rust
  impl Summary for Tweet {
      fn summarize_author(&self) -> String {
          format!("@{}", self.username)
      }

      // We do not require to define the summarize() method
      // as we can use the trait's default implementation
  }
  ```

Note: Calling the default implementation from an overriding implementation won't work.

#### Traits as Paramteres

- You can define the `type` of parameters of a function as a trait.
- Then, you can pass any type that implements the specified trait.
- Here's the syntax for that:

  ```rust
  pub fn notify(item: &impl Summary) {
      println!("Breaking news! {}", item.summarize());
  }
  ```

- Code that calls the function with any other type, such as a `String` or an `i32`, won’t compile because those types don’t implement Summary.

- The above syntax is the simpler version of this original syntax, known as "_trait bound syntax_":

  ```rust
  pub fn notify<T: Summary>(item: &T) {
      println!("Breaking news! {}", item.summarize());
  }
  ```

- It is possible to use this syntax like this:

  ```rust
  pub fn notify<T: Summary>(item1: &T, item2: &T) {
  ```

- It is possible to define multiple trait bounds for a single parameter:

  ```rust
  // In both these cases, item must be a type that
  // implements both traits Summary and Display

  // Method 1 ->
  pub fn notify(item: &(impl Summary + Display)) {

  // Method 2 ->
  pub fn notify<T: Summary + Display>(item: &T) {
  ```

- You can use `where` clause to declutter the signature. For Example:

  ```rust
  fn some_function<T, U>(t: &T, u: &U) -> i32
      where T: Display + Clone,
            U: Clone + Debug
  {
  ```

- Similar to function parameters, it is possible to return types that implements traits:

  ```rust
  // Signature says that it'll return any type that implements the trait Summary
  fn returns_summarizable() -> impl Summary {
      // Tweet is some type that implements Summary
      Tweet {
          username: String::from("horse_ebooks"),
          content: String::from(
              "of course, as you probably already know, people",
          ),
          reply: false,
          retweet: false,
      }
  }
  ```

- However, you can only use `impl Trait` if you’re returning a single type. For example:

  ```rust
  // FAIL: Either return NewsArticle or Tweet (only one type that implements Summary)
  fn returns_summarizable(switch: bool) -> impl Summary {
      if switch {
          NewsArticle {
            ...
          }
      } else {
          Tweet {
            ...
          }
      }
  }
  ```

- Finding the largest character of an array of integer or an array of characters using generics and traits:

  ```rust
  // Generic is used as we defined T in the signature, allowing any type to pass
  // Trait bound is specified as PartialOrd is added to the signature, allowing any type that allows comparison, and copy (both i32 and char do)
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
      let mut largest = list[0];

      for &item in list {
          if item > largest {
              largest = item;
          }
      }

      largest
  }

  fn main() {
      let number_list = vec![34, 50, 25, 100, 65];

      let result = largest(&number_list);
      println!("The largest number is {}", result);

      let char_list = vec!['y', 'm', 'a', 'q'];

      let result = largest(&char_list);
      println!("The largest char is {}", result);
  }
  ```

- Using Trait Bounds to Conditionally Implement Methods:

  ```rust
  use std::fmt::Display;

  struct Pair<T> {
      x: T,
      y: T,
  }

  impl<T> Pair<T> {
      fn new(x: T, y: T) -> Self {
          Self { x, y }
      }
  }

  // cmp_display will only run on types bounded by traits Display and PartialOrd, hence works conditionally
  impl<T: Display + PartialOrd> Pair<T> {
      fn cmp_display(&self) {
          if self.x >= self.y {
              println!("The largest member is x = {}", self.x);
          } else {
              println!("The largest member is y = {}", self.y);
          }
      }
  }
  ```
