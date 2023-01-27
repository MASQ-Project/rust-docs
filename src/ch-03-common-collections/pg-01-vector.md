## Vector

- It is represesnted as `Vec<T>`.
- You can store variable number of values, unlike Array. Though, the data type of stored values should be same.
- Vectors store values next to each other in memory.
- Creating a new vector:

  ```rust
    // We'll add type annotation because the vector has 0 elements,
    // hence, there is no way for Rust to recognize type implicitly
    let v: Vec<i32> = Vec::new();
  ```

- Creating vectors using a macro:

  ```rust
    let v = vec![1, 2, 3];
  ```

- Pushing new values (make sure Vector is mutable):

  ```rust
    v.push(5);
  ```

- Popping new values:

  ```rust
  let mut vec = vec![1, 2, 3];
  assert_eq!(vec.pop(), Some(3));
  ```

- Even though vectors store values on heap and it is possible to introduce references to the elements of the vector. Still, the vectors automatically cleans up memory as it goes out of scope:

  ```rust
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
  ```

- Accessing elements inside a vector:

  - Method 1:

    ```rust
      let third: &i32 = &v[2]; // Might panic due to out of index
    ```

  - Method 2:

    ```rust
    match v.get(2) { // Gives Option<&T>
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
    ```

- You can't do that:

  ```rust
      let mut v = vec![1, 2, 3, 4, 5];

      let first = &v[0]; // A reference to immutable vector [Immutable Borrow]

      v.push(6); // Writing to a mutable vector [Mutable Error]

      println!("The first element is: {}", first); // Accessing the reference after writing [Immutable Borrow Used]

      // Recall: You can’t have mutable and immutable references in the same scope.
  ```

- Why should a reference to the first element care about what changes at the end of the vector? This error is due to the way vectors work: adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space, if there isn’t enough room to put all the elements next to each other where the vector currently is.

- Itearting over the Vector:

  ```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{}", i);
    }
  ```

- Iterating and mutating the vector:

  ```rust
      let mut v = vec![100, 32, 57];
      for i in &mut v {
          *i += 50;
      }
  ```

- Storing values of different types using enums:

  ```rust
      // Using enums in vectors add stability as
      // when we'll use match all possible cases
      // will be covered.
      enum SpreadsheetCell {
          Int(i32),
          Float(f64),
          Text(String),
      }

      let row = vec![
          SpreadsheetCell::Int(3),
          SpreadsheetCell::Text(String::from("blue")),
          SpreadsheetCell::Float(10.12),
      ];
  ```
