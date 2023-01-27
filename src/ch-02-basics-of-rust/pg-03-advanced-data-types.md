## Advanced Data Types

#### Strings

- They are represented in three types:

  - `String` - A smart pointer.
  - `&String` - Reference to a String.
  - `&str` - String Slice

  - Defining a String

    ```rust
    let string = String::new("127.0.0.1:8080"); // It can grow and shrink
    let string_literal = "1234"; // It's memory is fixed at runtime
    ```

  - Slicing a string

    ```rust
    let string = String::from("127.0.0.1:8080");
    let string_slice = &string[10..14]; // We can also use &string[10..]

    // We can also use
    let string_slice = &string[10..]; // Give me everything after 10th byte not character
    let string_slice = &string[..12]; // Give me everything upto 12th byte not character
    ```

  - Rust uses UTF-8 encoding. So, prefer not to not pass integer values for slicing, as the slice function slices on the basis of bytes instead of characters.

    ```rust
    let string = String::from("😀😃😄😁");
    let string_slice = &string[..4];
    ```

    For this slice instead of returning 4 emojis, the rust will return 1 emoji because it takes 4 bytes to store an emoji.

    ```zsh
    string_slice = "😀"
    ```

  - Strings in rust can dynamically grow or shrink.

  - We can borrow an entire string by using this syntax

    ```rust
    let string = String::from("127.0.0.1:8080");
    let string_borrow: &str = &string;
    ```

- This is how `let s1 = String::from("hello");` is stored in Rust:

  - Left - Parts of String that are stored on the stack:
    1. Pointer: Points to the memory that holds the contents of the string.
    2. Length: How much memory, in bytes, the contents of the String is currently using.
    3. Capacity: The total amount of memory, in bytes, that the String has received from the allocator.
  - Right - The memory on the heap that holds the contents.

  ![Image](https://doc.rust-lang.org/book/img/trpl04-01.svg)

## String Slicing

- Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection.
- A slice is a kind of reference, so it does not have ownership.
- Slices are represented by `&str` and are _immutable_.

  ```rust
      let s = String::from("hello world");

      let hello = &s[0..5]; // or you can use &s[..5]
      let world = &s[6..11];
  ```

- This will throw compile-time error:

  ```rust
  // FAIL: You cannot clear the memory, to which some reference already exists
  fn main() {
      let mut s = String::from("hello world");

      let word = first_word(&s); // Returns a &str, which is a referenc to s

      s.clear(); // error!

      println!("the first word is: {}", word);
  }
  ```

- Thus, String Slices helps us write secure code by protecting the references to a string.
- Also string literals `let string_literal = "hello";`, are string slices `&str`, and are immutable.

Note: It is expected that the String only contains ASCII characters, because in case of UTF-8, if we try to slice between a multibyte character, it'll cause an error.

- The correct way to use referencing is discussed int the section, ["Which is better `&String` or `&str`?"](#which-is-better-string-or-str).

- Difference between String, String Literal, String Slice

  | Property          | String                                      | String Literal                 | String Slice                       | Reference to String              |
  | ----------------- | ------------------------------------------- | ------------------------------ | ---------------------------------- | -------------------------------- |
  | Definition        | `let string = String::from("some_string");` | `let string_literal = "1234";` | `let string_slice = &string[1..3]` | `let string_reference = &string` |
  | Representation    | `String`                                    | `&str`                         | `&str`                             | `&String`                        |
  | Mutable           | :white_check_mark:                          | :x:                            | :x:                                | :x:                              |
  | Memory Management | Heap (but deallocates when out of scope)    | Heap (Points to binary)        | Heap (Points to Binary)            | Heap                             |
  | Use Cases         | Taking Input, or any String Manipulation    | Defining Constant Strings      | Slicing and Borrowing              | Borrowing                        |

#### Strings and UTF-8 encoding

- Characters are represented by single inverted commas, and has `4 bytes` of storage. For Example, `'😀'`.
- String is not a collection of characters but collections of bytes.
- Rust has only one string type in the core language, which is the string slice `str` that is usually seen in its borrowed form `&str`.
- String Slices are the references to some UTF-8 data stored somewhere else.
- String Literals are string slices when stored in program's binary.
- The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type.
- When Rustaceans, call "string in rust", they collectively mean:
  - `String`
  - `&str`
- Both `String` and `&str` are UTF-8 encoded.
- Creating the `String` type:

  ```rust
  let mut s = String::new();
  ```

- To create a `String` from some starting string:

  ```rust
  let s = "initial contents".to_string(); // This fn can be used on any type that implements Display trait
  let s = String::from("initial contents");
  ```

- It is possible to store any properly encoded data:

  ```rust
  let hello = String::from("नमस्ते");
  let hello = String::from("안녕하세요");
  let hello = String::from("Здравствуйте");
  ```

- Updating the String:

  ```rust
  let mut s = String::from("foo");
  s.push_str("bar"); // It takes string slice, hence doesn't takes ownership
  s.push('!'); // This fn only takes character as argument.
  // s will become "foobar!"
  ```

- Concatenating two strings with the `+` operator:

  ```rust
  // '+' is a replacement of - fn add(self, s: &str) -> String {
  let s1 = String::from("Hello, ");
  let s2 = String::from("world!");
  let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
  ```

Note: In Rust, if we provide `&str`, as a function's argument, it can accept both `&String` and `&str`. Rust uses a deref coercion, which here turns `&s2` into `&s2[..]`.

- Combining multiple strings or formatting them:

  ```rust
  let s1 = String::from("tic");
  let s2 = String::from("tac");
  let s3 = String::from("toe");

  // Method 1
  let s = s1 + "-" + &s2 + "-" + &s3;

  // Method 2
  let s = format!("{}-{}-{}", s1, s2, s3); // It works like println!() but returns String
  ```

- Indexing into Strings is not possible and results in error:

  ```rust
  // FAIL: Strings can be indexed in Rust
  let s1 = String::from("hello");
  let h = s1[0]; // Won't work
  ```

- How values are stored in string.

  - String is just a wrapper over `Vec<u8>`, this means `1 byte` of space for each element in the vector. Hence, if we want to save special charcters, then it may take more than one element to store the values.
  - Let's consider following examples:

    ```rust
    let hello = String::from("Hola"); // Each character will take 1 byte of storage
    let hello = String::from("Здравствуйте"); // Each character will take 2 bytes of storage
    ```

  - Let's understand using the Hindi word `“नमस्ते”`:

    - As Bytes (the way `String` does using `u8` which ranges from `0` to `255`):

      ```rust
      [224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
      224, 165, 135]
      ```

    - As Unicode Scalar Values (the way `char` does):

      ```rust
      ['न', 'म', 'स', '्', 'त', 'े']
      ```

    - As Grapheme Clusters (the way a Hindi speaker might do):

      ```rust
      ["न", "म", "स्", "ते"]
      ```

- Slicing Strings:

  - You need to provide the range of `bytes` to be sliced out of String. Again, not characters but bytes.

    ```rust
    let hello = "Здравствуйте"; // Each character here is composed of 2 bytes

    let s = &hello[0..4]; // It'll save first 4 bytes, `Зд`

    let will_panic = &hello[0..1]; // It'll panic, as if invalid index was accessed in the vector.
    ```

- Iterating over strings:

  - You can iterate over the unicode scalar values or what `chars` might store:

    ```rust
    for c in "नमस्ते".chars() {
        println!("{}", c);
    }

    // This is what it'll print
    न
    म
    स
    ्
    त
    े
    ```

  - You can iterate over bytes also, the way `String` is stored in `Vec<u8>` format:

    ```rust
    for b in "नमस्ते".bytes() {
        println!("{}", b);
    }

    // The output will be like
    224
    164
    // --snip--
    165
    135
    ```