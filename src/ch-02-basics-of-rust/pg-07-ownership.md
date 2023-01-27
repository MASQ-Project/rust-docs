## Ownership

- _Ownership_ is a set of rules that governs how a Rust program manages memory.
- Some languages have garbage collection that constantly looks for no-longer used memory as the program runs; in other languages, the programmer must explicitly allocate and free the memory.
- Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks.
- If any of the rules are violated, the program won’t compile.
- None of the features of ownership will slow down your program while it’s running.

### The stack and the heap

- Both the stack and the heap are parts of memory available to your code to use at runtime

- Differences between Stack and Heap

| Stack                                                          | Heap                                                                                       |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Value is stored in order _Last In, First Out_.                 | Value is stored at an empty spot and a _pointer_ is returned.                              |
| More Organized                                                 | Less Organized                                                                             |
| Operations are _push_ and _pop_.                               | Process of storing data on heap is called allocating.                                      |
| All stored data is of fixed size.                              | Stored data can be of dynamic size.                                                        |
| New items are stored on top of stack, hence pushing is faster. | New items are stored after searching for right place to store, hence allocating is slower. |

- Pushing values onto the stack is not considered allocating. Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer.
- Think of heap as being seated at a restaurant.

  - When you enter, you state the number of people in your group, and the staff finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.
  - Consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap). Allocating a large amount of space on the heap can also take time.

- When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

### Ownership Rules

- These are three golden rules of ownership:

  1. Each value in Rust has a variable that’s called its _owner_.
  2. There can only be one owner at a time.
  3. When the owner goes out of scope, the value will be dropped.

### What is `moved`?

- This is not a move, it's a copy:

```rust
// y = x is a copy: Integers are simple values with a known, fixed size, pushed to stack, hence copied
let x = 5;
let y = x;
```

- This is a move:

```rust
// s1 = s2 is a move: Strings are stored on heap, hence only the data of string stored on stack is copied, hence moved
let s1 = String::from("hello");
let s2 = s1;
```

- Why `String` is only moved and not copied?

  - When we assign s1 to s2, the String data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack.
  - We do not copy the data on the heap that the pointer refers to.
  - Hence it is moved not copied.

  ![Move of String](https://doc.rust-lang.org/book/img/trpl04-02.svg)

- Why Rust preferes moving instead of copying the heap data?

  - If Rust preformed copy instead of move, the operation `s2 = s1` could be very expensive in terms of runtime performance if the data on the heap were large.
  - This is what it would look like if Rust would have copied instead of moved.

  ![If String was Copied](https://doc.rust-lang.org/book/img/trpl04-03.svg)

- How does Rust clean memory after we perform `s2 = s1` and both `s1` and `s2` go out of scope?

  - When a variable saved on heap goes out of scope, Rust calls a `drop` function to clean it from the memory.
  - But we performed `let s2 = s1;`, so Rust will try to clean both `s1` and `s2`, cleaning the same memory.
  - This problem is called _double free_ error.
  - To solve this problem, when we perform `let s2 = s1;`, Rust actually moves the value to `s2` by invalidating `s1`.
  - Now, Rust has to only clean the `s2` variable. Hence, the problem of double free is solved.
  - So, what may look like a shallow copy (refer shallow and deep copy in other languages), it is actually a move operation.
    ![Invalidation of s1](https://doc.rust-lang.org/book/img/trpl04-04.svg)

- In addition, there’s a design choice that’s implied by this: Rust will never automatically create “deep” copies of your data.
- Therefore, any automatic copying can be assumed to be inexpensive in terms of runtime performance.

### Clone

- This function is used when we want to clone the heap data.
- If we perform `s2 = s1`, on a heap data for example String, then only the stack data will be copied and not the heap data, hence moved.
- In case we want to copy the heap data too (also referred to deep copy), we use _clone_ function.
- Here's an example:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

### The `Copy` and `Drop` trait

- `Copy` trait can be placed on types that are stored on the stack like integers are.

- If a type implements the Copy trait, a variable is still valid after assignment to another variable.
- Rust won’t let us annotate a type with Copy if the type, or any of its parts, has implemented the Drop trait.
- If the type needs something special to happen when the value goes out of scope and we add the Copy annotation to that type, we’ll get a compile-time error.
- Types that implement copy:
  1. All the integer types, such as u32.
  2. The Boolean type, bool, with values true and false.
  3. All the floating point types, such as f64.
  4. The character type, char.
  5. Tuples, if they only contain types that also implement Copy. For example, (i32, i32) implements Copy, but (i32, String) does not.

### Ownership and Functions

- Passing a variable to a function will move or copy, just as assignment does.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

- This is how the ownership works for the return values:

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return value into s1
    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into takes_and_gives_back, which also moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String { // gives_ownership will move its return value into the function that calls it
    let some_string = String::from("yours"); // some_string comes into scope

    some_string // some_string is returned and moves out to the calling function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into scope
    a_string  // a_string is returned and moves out to the calling function
}
```

- When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless ownership of the data has been moved to another variable. See how `takes_and_gives_back` function returns the variable before going out of scope.
- Basically if we send a variable, we must return it back from the function to use it again.
- So, there are two things we can do, either _return multiple values using tuples_ or use _references_.

```rust
// This is an example of how a fn returns multiple values using tuples
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```
