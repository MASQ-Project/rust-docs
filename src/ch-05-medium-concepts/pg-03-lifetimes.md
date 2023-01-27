## Lifetimes

Fun Fact: The developers who are programming Rust are constantly programming the patterns into the compiler’s code so the borrow checker could infer the lifetimes in some situations and wouldn’t need explicit annotations. These patterns programmed into Rust’s analysis of references are called the _lifetime elision rules_. Thus, making lifetimes easier to use day by day.

- Lifetime is a way to specify how long the multiple references will live. So, it doesn't make sense to add lifetime to just one reference, they must be multiple.
- Ways to add lifetime specifiers:

  ```rust
  &i32        // a reference
  &'a i32     // a reference with an explicit lifetime
  &'a mut i32 // a mutable reference with an explicit lifetime
  ```

Note: We'll may or may not use lifetimes only when we're dealing with references.

- For example, let’s say we have a function with the parameter `first` that is a reference to an `i32` with lifetime `'a`. The function also has another parameter named `second` that is another reference to an `i32` that also has the lifetime `'a`. The lifetime annotations indicate that the references `first` and `second` must both live as long as that generic lifetime.

- _Every reference_ in Rust has a _lifetime_.
- Here's an exmple of dangling reference:

  ```rust
  // FAIL: Rust prevents dangling references
      {
          let r;                // ---------+-- 'a
                                //          |
          {                     //          |
              let x = 5;        // -+-- 'b  |
              r = &x;           //  |       |
          }                     // -+       | <- x dies but r stores reference of x, hence r stores a dangling referece
                                //          |
          println!("r: {}", r); //          |
      }                         // ---------+
  ```

- Rust won't compile the above code, as it uses a `borrow checker`, to verify whether a reference or borrow is valid or not.
- We may fix it by fixing the lives of variables by declaring them at different places:

  ```rust
      {
          let x = 5;            // ----------+-- 'b
                                //           |
          let r = &x;           // --+-- 'a  |
                                //   |       |
          println!("r: {}", r); //   |       |
                                // --+       |
      }                         // ----------+
  ```

- This code will not compile, it'll require lifetime specifiers:

  ```rust
  // FAIL: Rust can’t tell whether the reference being returned refers to x or y.
  fn longest(x: &str, y: &str) -> &str {
      if x.len() > y.len() {
          x
      } else {
          y
      }
  }
  ```

  ```rust
  // Compiler will ask us to rewrite the signature like this
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  // Lifetimes on function or method parameters are called input lifetimes, and lifetimes on return values are called output lifetimes.
  ```

- The reason why Rust asks us to specify the lifetimes are due to these reasons:

  - We don’t know whether the if case or the else case will execute.
  - We also don’t know the concrete lifetimes of the references that will be passed in.

- When we add the lifetime specifiers as specified by the compiler, it means, the generic lifetime `'a` will get the concrete lifetime that is equal to the smaller of the lifetimes of `x` and `y` (the variables passed in).

Note: Remember, when we specify the lifetime parameters in this function signature, we’re not changing the lifetimes of any values passed in or returned. Rather, we’re specifying that the borrow checker should reject any values that don’t adhere to these constraints. Note that the longest function doesn’t need to know exactly how long `x` and `y` will live, only that some scope can be substituted for `'a` that will satisfy this signature.

- How borrow checker will allow:

```rust
// Works: result is valid until the inner scope ends, string2 and string1 are valid too, hence borrow checker allows
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}

// FAILS: The way we've specified lifetimes, result should have a shorter lifetime, equivalent to that of string2. Since, the code doesn't follows the rule, it fails.
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

- In the second case, this is the error that the compiler will throw:

```zsh
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here
```

- The below code will not compile because even though we’ve specified a lifetime parameter `'a` for the return type, this implementation will fail to compile because the return value lifetime is not related to the lifetime of the parameters at all.

  ```rust
  fn longest<'a>(x: &str, y: &str) -> &'a str {
      let result = String::from("really long string");
      result.as_str()
  }
  ```

- The compiler will throw this error, since Rust will prevent us from creating dangling reference.

  ```zsh
    --> src/main.rs:11:5
     |
  11 |     result.as_str()
     |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function
  ```

- In this case, the best fix would be to return an owned data type rather than a reference so the calling function is then responsible for cleaning up the value.

- Rust is improving day by day to not require programmers to use lifetimes in some places. For Example:

  ```rust
  // Even though we're dealing with references in functions,
  // compiler won't ask us to specify lifetimes, it's because
  // rust devs improved the rust compiler so that the borrow
  // checker need not to not ask for lifetimes in this case
  fn first_word(s: &str) -> &str {
      let bytes = s.as_bytes();

      for (i, &item) in bytes.iter().enumerate() {
          if item == b' ' {
              return &s[0..i];
          }
      }

      &s[..]
  }

  // In earlier version (pre-1.0), the signature would've looked like this
  fn first_word<'a>(s: &'a str) -> &'a str {
  ```

#### Rules of lifetimes

- There are 3 rules that compiler follows to verify whether lifetimes are valid or not.

  - **First Rule:** _Each parameter that is a reference gets its own lifetime parameter._
    - A function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32)`;
    - A function with two parameters gets two separate lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32);` and so on.
  - **Second Rule:** _If there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters._
    - For Example, `fn foo<'a>(x: &'a i32) -> &'a i32`.
    - There was only one parameter, hence one lifetime for inputs, so the same lifetime was assigned to the output.
  - **Third Rule:** _If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters._
    - This third rule makes methods much nicer to read and write because fewer symbols are necessary.
    - Please note that this rule only applies to methods (functions that uses `self`), and not to simple functions.

- You can read in detail about How compiler automatically applies lifetimes and the about the rules of lifetimes in [_Lifetime Elision_](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision).

#### Lifetimes in Structs and Methods

- Lifetimes in struct. It’s possible for structs to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition.

  ```rust
  struct ImportantExcerpt<'a> {
      // Since, string slice is a referece, we added lifetime, such that field part and struct lives together
      part: &'a str,
  }

  fn main() {
      let novel = String::from("Call me Ishmael. Some years ago...");
      let first_sentence = novel.split('.').next().expect("Could not find a '.'");
      let i = ImportantExcerpt {
          part: first_sentence,
      };
  }
  ```

- Lifetimes in `impl` blocks:

  ```rust
  // The lifetime parameter declaration after impl and its use after the type name are required,
  // but we’re not required to annotate the lifetime of the reference to self because of the first elision rule.
  impl<'a> ImportantExcerpt<'a> {
      // No need to apply in the method below due to the first elision rule
      fn level(&self) -> i32 {
          3
      }
      // No need to apply in the method below due to the third elision rule
      fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```

#### The static lifetime

- The `'static` is a lifetime which means that this reference can live for the entire duration of the program.
- All string literals have the `'static` lifetime.
- You may use it as shown in the code below:

  ```rust
  let s: &'static str = "I have a static lifetime.";
  ```

Note: You might see suggestions to use the `'static` lifetime in error messages. But before specifying `'static` as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not. You might consider whether you want it to live that long, even if it could. Most of the time, the problem results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is fixing those problems, not specifying the `'static` lifetime.


### Generic Type Parameters, Trait Bounds, and Lifetimes Together

- You may consider the code below, it prints the type (they type `T` can be filled with any type that implements `Display` trait), also it returns the longest string slice.

```rust
// Generic Type: T
// Trait Bounds: Display
// Lifetime: 'a
use std::fmt::Display;

// Because lifetimes are a type of generic, the declarations of
// the lifetime parameter 'a and the generic type parameter T go
// in the same list inside the angle brackets after the function name.
fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```