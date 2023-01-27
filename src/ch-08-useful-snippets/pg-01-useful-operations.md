## Useful operations

- Taking Input and performing type conversions

  ```rust
  // Always import when you want to take input from user
  use std::io;

  fn main() {
    let mut input = String::new();

    // read_line() returns io::Result, which is an enum
    // It acts as a match, either returns Ok() with value or prints error.
    io::stdin().read_line(&mut input)
               .expect("Failed to read line.");

    // Declaring a variable with same name again is called shadowing, mostly used for type conversions.
    // Trims whitespaces at start and end, `/n` (newline) and `/r/n` (windows carriage return and a newline)
    // parse() (returns Result), parses the string to a number, into the defined type
    let input: u32 = match guess.trim()
                                .parse()
                                .expect("Invalid Input");

    println!("Your input: {}", input);
  }
  ```

- Generating Random Numbers (need external crate `rand`)

  ```rust
  // The Rng trait defines the functions that we'll use to generate random numbers
  use rand::Rng;

  fn main() {
    // thread_rng() is a random number generator that is local to the current thread of execution and seeded by the operating system.
    // gen_range() is a function part of Rng and it generates a random number of range [inclusive, exclusive), 1..101 === 1..=100
    let random_number = rand::thread_rng().gen_range(1..101);
  }
  ```

- A hack to know type definitions, is to write a wrong definition, and read the error from logs, they'll tell you the correct type.

  ```rust
  // FAIL: It doesn't have u32 as type, we wrote wrong type,
  // to find the correct one by reading error logs.
  let f: u32 = File::open("hello.txt");
  ```

- To read the contents of a file into a string:

  ```rust
  use std::fs;
  use std::io;

  fn read_username_from_file() -> Result<String, io::Error> {
      fs::read_to_string("hello.txt")
  }
  ```

- To return the last character of first line in a text.

  ```rust
  fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines() // Converts string slice into iterator of lines
        .next()? // Returns First element of iterator or None
        .chars() // Converts string slice into iteratot of characters
        .last() // Returns last element
  }
  ```

- Read command line arguments and store them into a vector.

  ```rust
  use std::env;

  fn main() {
      let args: Vec<String> = env::args().collect();
      println!("{:?}", args);
  }
  ```
