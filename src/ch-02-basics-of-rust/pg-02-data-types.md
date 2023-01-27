## Data Types

- Every value in Rust is of a certain _data type_, that makes Rust a _statically typed language_.
- It means Rust needs to know the type of all variables at compile time.
- Rust data types has two subsets:
  1. Scalar
  2. Compound

### Scalar Data Types

- Booleans
  - Represented by `bool`, has two values `true` and `false`.
- Characters
  - Represented by `char`, it always has space of `4` bytes or `32` bits instead of 1 byte.
  - Characters are UTF-8 encoded, thus supports `'z', 'ℤ', '😻'`.
  - Characters use single quotes and string uses double quotes.
- Integers

  - `u` means unsigned (only positive), `i` means signed (both positive & negative)
  - The size ranges from `8` bits to `128` bits.
  - Range of Unsigned Integers is:
    $$ [0, 2^n - 1] $$
  - Range of Signed Integer is:
    $$ [- 2^{n-1}, 2^{n-1} - 1]$$
  - Examples, `u8`, `i8`, `u128`, `i128`
  - Rust also supports, `usize` and `isize` which means that it will take up space according to the architecture whether it is 32 bit or 64 bit.
  - You may also represent integer literals in the below forms:

| Number literals | Example | Integer |
| ----------- | ----------- | ----------- |
| Decimal | 98_222 | 98222 |
| Hex | 0xff | 255 |
| Octal | 0o77 | 63 |
| Binary | 0b1111_0000 | 240 |
| Byte (u8 only) | b'A' | 65 |


  - Integer types default to `i32`.
  - To read how Interger Overflow works in Rust, please follow [this link](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-overflow).
  - Division of integers gives floored value, `3 / 2 == 1`.

- Floats
  - It has `f32` and `f64`, two floating data types for size `32` and `64`.
  - Floating types default to `f64`.
  - Division of floats give fractional result, `3.0 / 2. 0 = 1.5`.

### Compound Data Types

- There are two compound data types in Rust:

  1. Tuples
  2. Arrays

#### Tuples

- They can store number of values with different data types.
- They can't grow or shrink once declared.
- Tuples can be declared as follows:

  ```rust
  let tup_with_types: (i32, f64, u8) = (500, 6.4, 1);
  let tup = (500, 6.4, 1);

  // Destructuring Tuples
  let (x, y, z) = tup;

  // Destructuing Tuples using .
  let x: (i32, f64, u8) = (500, 6.4, 1);
  let five_hundred = x.0;
  let six_point_four = x.1;
  let one = x.2;

  // Unit type tuple with unit value
  let unit_tup = ();
  ```

#### Array

- They can store number of values with same data type.
- They can't grow or shrink once declared as their memory is allocated on stack.
- If you want a similar data structure that can grow or shrink then use Vectors.
- If you access an index of array that is greater than it's length, it'll result in `'index out of bounds'`.
- In other low level languages, this check is not done and they return an invalid memory.
- Arrays can be declared as follows:

  ```rust
  // Simple array declaration
  let a = [1, 2, 3, 4, 5];

  // Declaring array with type and size
  let a: [i32; 5] = [1, 2, 3, 4, 5];

  // Declaring same value 3 for 5 elements
  let a = [3; 5];

  // Accessing Values of array
  let a = [1, 2, 3, 4, 5];
  let first = a[0];
  let second = a[1];
  ```

#### Slicing an array

- It is possible to slice an array in Rust.

  ```rust
  let a = [1, 2, 3, 4, 5];

  let slice = &a[1..3]; // It is of type &[i32]
  ```

#### Ranges

- You can create a range with `..` operator.
- The following are equal:
  - `1..5` ~ `1..=4`
  - `0..4` ~ `..4`
  - `1..len` ~ `1..`
  - `0..len` ~ `..`
