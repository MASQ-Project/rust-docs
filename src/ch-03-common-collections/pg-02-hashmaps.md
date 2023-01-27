## HashMaps

- The type `HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V`.
- It does this via a hashing function, which determines how it places these keys and values into memory.
- Creating a new HashMap:

  ```rust
      use std::collections::HashMap;

      let mut scores = HashMap::new();

      scores.insert(String::from("Blue"), 10);
      scores.insert(String::from("Yellow"), 50);
  ```

- Just like Vectors, HashMaps also save their values on Heap.
- All Keys must have same type, and all values must have same type.
- Creating HashMap through iterators:

  ```rust
      use std::collections::HashMap;

      let teams = vec![String::from("Blue"), String::from("Yellow")];
      let initial_scores = vec![10, 50];

      let mut scores: HashMap<_, _> =
          teams.into_iter()
               .zip(initial_scores.into_iter()) // creates a tuple, example ("Blue", 10)
               .collect(); // Converts tuple into HashMap
  ```

- HashMap and ownership: Types that implement `Copy` trait will be copied else moved. For Example, `i32` will be copied but `String` will be moved.

  ```rust
      use std::collections::HashMap;

      let field_name = String::from("Favorite color");
      let field_value = String::from("Blue");

      let mut map = HashMap::new();
      map.insert(field_name, field_value);
      // You can't use field_name or field_value now, as they've been moved
  ```

- Accessing value in HashMap, the `get` method returns `Option<&V>`:

  ```rust
      use std::collections::HashMap;

      let mut scores = HashMap::new();

      scores.insert(String::from("Blue"), 10);
      scores.insert(String::from("Yellow"), 50);

      let team_name = String::from("Blue");
      let score = scores.get(&team_name); // This is how we access value for a certain Key

      // The score variable will contain - Some(&10)
  ```

- Iterating over a HashMap:

  ```rust
      use std::collections::HashMap;

      let mut scores = HashMap::new();

      scores.insert(String::from("Blue"), 10);
      scores.insert(String::from("Yellow"), 50);

      for (key, value) in &scores {
          println!("{}: {}", key, value);
      }
  ```

- Updating a HashMap:

  - Overwriting the value:

    ```rust
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    // Output - scores = {"Blue": 25}
    ```

  - Only inserting the value if the Key has no value:

    ```rust
    scores.insert(String::from("Blue"), 10);

    // We'll need to use entry to use or_insert
    scores.entry(String::from("Yellow")).or_insert(50); // This will add 50
    scores.entry(String::from("Blue")).or_insert(50); // This won't replace 10 with 50

    // Output - scores = {"Yellow": 50, "Blue": 10}
    ```

  - Updating a value based on the Old Value:

    ```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0); // or_insert returns mutable reference to the Value, &mut V
        *count += 1;
    }

    println!("{:?}", map);

    // Output - map = {"world": 2, "hello": 1, "wonderful": 1}
    ```

- The hashing function that Rust uses is `SipHash`. You can replace the hashing function. Please [refer here](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hashing-functions) for more.
