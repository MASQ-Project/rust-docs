## Borrowing

- We do borrowing whenever we don't want to transfer the complete ownership of a varaible.
- Consider the above example in previous section, inside `calculate_length` function we returned String `s`.
- We returned it because when we passed `s1` into the function we transferred it's ownership to the variable `s`.
- Since, the scope of variable `s` is limited, the passed value of `s1` will die outside the scope of `calculate_length`.
- So, we returned the variable `s` inside a tuple before the end of it's scope.
- Now, there is a workaround to calculate length without transferring the ownership.
- The process of doing so is called _Borrowing_ and it's done using _references_.

### References

- A reference is a pointer to the variable.
- It’s an address we can follow to access data stored at that address that is owned by some other variable.
- Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type.
- We use `&`, called as ampersand, to represent a reference.

  ```rust
  fn main() {
      let s1 = String::from("hello");

      // Instead of transferring ownership, we only passed a reference to the string
      let len = calculate_length(&s1);

      println!("The length of '{}' is {}.", s1, len);
  }

  // We declare that this function will only accept a reference to a String, hence only borrows
  fn calculate_length(s: &String) -> usize {
      s.len()
  }
  ```

- Instead of Ownership transfer, borrowing looks like this, where `s` stores the reference.

  ![Borrowing](https://doc.rust-lang.org/book/img/trpl04-05.svg)

#### Dereference

- This is the opposite of reference. It is represented by `*`.
- It returns the value of a pointer.

#### Mutable Reference

- The references are also immutable by default.
- To make a reference mutable, we need to make both the declared variable and the reference mutable using `mut` keyword.

  ```rust
  fn main() {
      let mut s = String::from("hello"); // Step 2 -> Change variable to mutable

      change(&mut s); // Step 3 -> Pass the string as a mutable reference
  }

  fn change(some_string: &mut String) { // Step 1 -> Declare in fn definition, that it demands a mutable reference
      some_string.push_str(", world");
  }
  ```

### Referencing for strings

#### Which is better `&String` or `&str`?

- Short Answer: `&str`.
- Reason: It allows us to use the same function on both `&String` values and `&str` values.

  ```rust
  fn first_word(s: &String) -> &str { // This sucks, only allows &String

  fn first_word(s: &str) -> &str { // Rustaceans prefer this, since it allows both `&String` and `&str`.
  ```

- Basically, `&str` works for all types of references:

  ```rust
  fn main() {
      let my_string = String::from("hello world");

      // `first_word` works on slices of `String`s, whether partial or whole
      let word = first_word(&my_string[0..6]);
      let word = first_word(&my_string[..]);
      // `first_word` also works on references to `String`s, which are equivalent
      // to whole slices of `String`s
      let word = first_word(&my_string);

      let my_string_literal = "hello world";

      // `first_word` works on slices of string literals, whether partial or whole
      let word = first_word(&my_string_literal[0..6]);
      let word = first_word(&my_string_literal[..]);

      // Because string literals *are* string slices already,
      // this works too, without the slice syntax!
      let word = first_word(my_string_literal);
  }
  ```

### Data Race

- The _data race_ condition happens when these three behaviors occur:

  1. Two or more pointers access the same data at the same time.
  2. At least one of the pointers is being used to write to the data.
  3. There’s no mechanism being used to synchronize access to the data.

- To prevent this condition, Rust adds limitations while using references.

#### Limitations of Referecnes

- We cannot create two mutable references to a variable.

  ```rust
      let mut s = String::from("hello");

      let r1 = &mut s;
      let r2 = &mut s;

      println!("{}, {}", r1, r2);
  ```

- We cannot create one immutable and one mutable reference to a variable.

  ```rust
      let mut s = String::from("hello");

      let r1 = &s; // no problem
      let r2 = &s; // no problem
      let r3 = &mut s; // BIG PROBLEM

      println!("{}, {}, and {}", r1, r2, r3);
  ```

#### Allowed Workarounds for References

- Multiple immutable references are allowed because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

  ```rust
      let mut s = String::from("hello");

      let r1 = &s; // no problem
      let r2 = &s; // no problem

      println!("{}, {}", r1, r2);
  ```

- Rust treats last usage of a immutable reference, as it's end. Hence, the following code runs perfectly, read more about [Non-Lexical Lifetimes](https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html#non-lexical-lifetimes).

  ```rust
      let mut s = String::from("hello");

      let r1 = &s; // no problem
      let r2 = &s; // no problem
      println!("{} and {}", r1, r2);
      // variables r1 and r2 will not be used after this point

      let r3 = &mut s; // no problem
      println!("{}", r3);
  ```

- You may create a new scope to use two mutable references.

  ```rust
      let mut s = String::from("hello");

      {
          let r1 = &mut s;
      } // r1 goes out of scope here, so we can make a new reference with no problems.

      let r2 = &mut s;
  ```

### Dangling References

- Dangling Reference is a reference to a location in memory which is freed but the reference exists.
- In languages with pointers, it’s easy to erroneously create a dangling pointer.
- In Rust, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

- Compiler throws error if we manually try to create a dangling reference:

  ```rust
  // FAIL: Rust won't allow you to create Dangling References
  fn main() {
      let reference_to_nothing = dangle();
  }

  fn dangle() -> &String {
      let s = String::from("hello");

      &s
  } // s goes out of scope here, but we try to reference to
  ```

- The Error that compiler throws is:

  ```zsh
  this function's return type contains a borrowed value, but there is no value
  for it to be borrowed from
  ```

- It also mentions, we can fix it using lifetimes.

### Golden Rules of Referencing

1. [No Data Racing](#data-race) - At any given time, you can have either one mutable reference or any number of immutable references.
2. [No Dangling References](#dangling-references) - References must always be valid.
