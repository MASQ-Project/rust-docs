### Unsafe Rust

_Rust has a second language hidden inside it that doesn’t enforce the memory safety guarantees: it’s called **unsafe Rust** and works just like regular Rust, but gives us extra superpowers._

- Why it exists?

  - It’s better for Rust to reject some valid programs rather than accept some invalid programs.
  - That makes the static analysis of the Rust compiler conservative.
  - Although the code might be okay, if the Rust compiler doesn’t have enough information to be confident, it will reject the code.
  - In these cases, you can use unsafe code to tell the compiler, “Trust me, I know what I’m doing.”
  - Another reason Rust has an unsafe alter ego is that the underlying computer hardware is inherently unsafe. Hence, it'll allow you to write low-level systems code, such as directly interacting with the OS, or even write your own OS.

- Any Downsides?

  - Use it at your own risk.
  - Problems due to memory unsafety, such as null pointer dereferencing, can occur.

- Answers to some common misconceptions:

  - It’s important to understand that `unsafe` doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use a reference in unsafe code, it will still be checked.
  - Hence, you'll get only the above mentioned features along with some safety.
  - Also, it does not necessarily mean that code inside `unsafe` is necessarily dangerous or that it will definitely have memory safety problems.
  - It is programmer's responsibilty to ensure that the code is memory safe.

- How to write code safely using `unsafe`?

  - Keep unsafe blocks small and it'll be easier to investigate the memory bugs.
  - You can also wrap unsafe code in a safe abstraction. It prevents the uses of unsafe from leaking out in all the other places.

- What Superpowers can I get?
  - Dereference a raw pointer
  - Call an unsafe function or method
  - Access or modify a mutable static variable
  - Implement an unsafe trait
  - Access fields of `union`s

#### Dereferencing a Raw Pointer

- Raw Pointers are meant for unsafe rust and are similar to references. They are of two types:
  - `*const T`: Immutable Raw Pointer
  - `*mut T`: Mutable Raw Pointer
- Here `*` is not a dereference operator but a part of the type name.
- Unlike references and Smart Pointers, they break the following rules of Rust's safety:
  - They are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
  - Aren’t guaranteed to point to valid memory
  - Are allowed to be null
  - Don’t implement any automatic cleanup
- This is how you can create raw pointers out of a variable.

  ```rust
  let mut num = 5;

  let r1 = &num as *const i32;
  let r2 = &mut num as *mut i32;
  ```

- We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block.

  ```rust
  let mut num = 5;

  // Notice it's possible to create raw pointers inside safe code
  let r1 = &num as *const i32;
  let r2 = &mut num as *mut i32;

  // But to dereference a raw pointer you'll require an unsafe block
  unsafe {
      println!("r1 is: {}", *r1);
      println!("r2 is: {}", *r2);
  }
  ```

- We broke the Rust's safety measures, as we are able to use a mutable and immutable reference to a value. Now, as a programmer we made sure that these references are used properly inside the `unsafe` block.

- Uses of creating raw pointers:
  - Mostly used when interfacing with C code.
  - Calling an Unsafe Function or Method.
  - Building safe abstractions over unsafe code.

#### Call an unsafe function or method

- Defining an unsafe function:

  ```rust
  unsafe fn dangerous() {}
  ```

- Calling an unsafe function:

  ```rust
  // By calling an unsafe function within an unsafe block,
  // we’re saying that we’ve read this function’s documentation
  // and take responsibility for upholding the function’s contracts.
  unsafe {
      dangerous();
  }
  ```

##### Wrappping unsafe code in safe abstractions

- We want to create a function that can split a vector into two by index

  ```rust
  // FAIL: Rust won't allow us to have two immutable borrow of the same vector
  // Only we know that the two immutable borrow aren't overlapping and won't
  // cause any trouble so we would like to silent the compiler by using unsafe
  fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
      let len = slice.len();

      assert!(mid <= len);

      (&mut slice[..mid], &mut slice[mid..])
  }
  ```

- Here is it's implementation using unsafe

  ```rust
  use std::slice;

  // Notice the function isn't using unsafe in it's signature, hence unsafe is
  // wrapped in a safe abstraction
  fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
      let len = slice.len();
      let ptr = slice.as_mut_ptr(); // Raw pointer

      assert!(mid <= len);

      // This is an acceptable use of unsafe
      // We need unsafe block to call these functions because
      // we're slicing and adding to the raw pointers, which may
      // have a chance to become invalid, iff programmer hasn't
      // written in properly
      unsafe {
          (
              slice::from_raw_parts_mut(ptr, mid), // It will give the slice of range [ptr, ptr + mid)
              slice::from_raw_parts_mut(ptr.add(mid), len - mid),
          )
      }
  }
  ```

- If you want to create a raw pointer with unexpected behaviour, you can do this

  ```rust
  // FAIL: It won't point to a valid i32 value
  use std::slice;

  let address = 0x01234usize;
  let r = address as *mut i32;

  let slice: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
  ```

##### Call the code from other languages using `extern`

- Rust uses `extern` keyword to use _Foreign Function Interface (FFI)_, it is a way for a programming language to define functions and enable a different (foreign) programming language to call those functions.
- Functions declared within extern blocks **are always unsafe to call** from Rust code.
- This is how you can call `C` code in Rust:

  ```rust
  // After extern you need to specify ABI (Application Binary Interface)
  // Here we are using extern to use ABI of other languages
  extern "C" {
      fn abs(input: i32) -> i32;
  }

  fn main() {
      unsafe {
          println!("Absolute value of -3 according to C: {}", abs(-3));
      }
  }
  ```

- It is possible to write Rust code such that other languages can call.

  ```rust
  // This code is not unsafe
  #[no_mangle] // This doesn't allows the compiler to rename the functions name
  pub extern "C" fn call_from_c() {   // Here we are using extern to create an ABI for other languages
      println!("Just called a Rust function from C!");
  }
  ```

#### Accessing or Modifying a Mutable Static Variable

- In Rust, **global** variables are called _static variables_.
- It is problematic as it may cause a data race if two threads are accessing the same mutable global variable.
- This is how you can create a global or static variable.

  ```rust
  static HELLO_WORLD: &str = "Hello, world!";

  fn main() {
      println!("name is: {}", HELLO_WORLD);
  }
  ```

- The references for static variable is `'static` by default. So, we need to specify it's lifetime anywhere.
- Also, it's completely safe to access an immutable static variable.

##### Constants and Static Variable

| Constants                                     | Static Variable                                                                                                            |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Dynamic address in memory                     | Fixed address in memory                                                                                                    |
| Constants duplicate their data whenever used. | Using the value will always access the same data.                                                                          |
| Constants are never mutable.                  | Static variable can be both mutable and immutable, and for modifying mutable static variable, you'll need to use `unsafe`. |

##### Implementing Static Variables

- This is how you can implement static variables.

  ```rust
  static mut COUNTER: u32 = 0;

  fn add_to_count(inc: u32) {
      unsafe {
          COUNTER += inc;
      }
  }

  fn main() {
      add_to_count(3);

      unsafe {
          println!("COUNTER: {}", COUNTER);
      }
  }
  ```

- Notice that, it's not causing us any trouble because this code is single threaded, but if we tried to mutate the static variable in multiple threads it could lead to data races.
- Static Variables (_or Global Variables_) are unsafe. That's because it's difficult to ensure that there are no data races for a global variable.
- It’s preferable to use the concurrency techniques and thread-safe smart pointers.

#### Implementing an unsafe trait

- A trait is unsafe when at least _one of its methods_ has some invariant that the compiler can’t verify.

- Here's an implementation:

  ```rust
  unsafe trait Foo {
      // methods go here
  }

  unsafe impl Foo for i32 {
      // method implementations go here
  }

  fn main() {}
  ```

- If we implement a type that contains a type that is not `Send` or `Sync` (i.e. doesn't already implements the safe ways of sending a type in multiple threads), such as _raw pointers_, and we want to mark that type as `Send` or `Sync`, we must use unsafe.

#### Accessing Fields of a Union

- A `union` is similar to a `struct`, but only one declared field is used in a particular instance at one time.
- Unions are primarily used to interface with unions in C code.
- Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance.
