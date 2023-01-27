## Pattern Matching

### The `match` conftrol flow operator

- It allows you to compare a value against a series of patterns and then execute code based on which pattern matches.
- It is possible to express very different kind of patterns. Also, Rust has a cumpolsary check, where it handles that all possible cases are handled.
- Think of a `match` expression as being like a coin-sorting machine: coins slide down a track with variously sized holes along it, and each coin falls through the first hole it encounters that it fits into.
- At the first pattern the value “fits”, the value falls into the associated code block to be used during execution.
- The expression with `if` statement only returns a boolean value but `match` expression can return any type.
- Here's an Example Below:

  ```rust
  enum Coin {
      Penny,
      Nickel,
      Dime,
      Quarter,
  }

  fn value_in_cents(coin: Coin) -> u8 {
      match coin {
          Coin::Penny => {
              println!("Lucky penny!");
              1
          },
          Coin::Nickel => 5,
          Coin::Dime => 10,
          Coin::Quarter => 25,
      }
  }
  ```

- Each new pattern under `match` is an arm. An arm has two parts: a pattern and some code.
- The code associated with each arm is an expression, and the resulting value of the expression in the matching arm is the value that gets returned for the entire match expression.

#### An `enum` inside another `enum`

- This is how we'll be using `match` for such cases:

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
  let _value = value_in_cents(Coin::Quarter(UsState::Alaska));
}
```

#### Matching with `Option<T>`

- The value inside Option of type `T` can be passed through like a functional argument using the `match` expression.

  ```rust
      fn plus_one(x: Option<i32>) -> Option<i32> {
          match x {
              None => None,
              Some(i) => Some(i + 1),
          }
      }

      let five = Some(5);
      let six = plus_one(five);
      let none = plus_one(None);
  ```

- The `match` expression always covers all the possible values, that's why we call them _exhaustive_: we must exhaust every last possibility in order for the code to be valid.

  ```rust
  // FAIL: All the cases not covered in match expression, the None case is remaining
  fn plus_one(x: Option<i32>) -> Option<i32> {
      match x {
          Some(i) => Some(i + 1),
      }
  }
  ```

- Especially in the case of `Option<T>`, when Rust prevents us from forgetting to explicitly handle the `None` case, it protects us from assuming that we have a value when we might have `null`, thus _making the billion-dollar mistake discussed earlier impossible_.

#### Catch remaining patterns using `_` placeholder

- It is possible to cover the remaining cases inside the `match` expression, it is similar to `default` case of `switch` statement in other languages.

  ```rust
      let dice_roll = 9;
      match dice_roll {
          3 => add_fancy_hat(),
          7 => remove_fancy_hat(),
          other => move_player(other), // This will match all the cases that aren't specifically listed
      }

      fn add_fancy_hat() {}
      fn remove_fancy_hat() {}
      fn move_player(num_spaces: u8) {}
  ```

- Sometimes we use placeholder `_`, to specify Rust, that this value is useless.

  ```rust
      let dice_roll = 9;
      match dice_roll {
          3 => add_fancy_hat(),
          7 => remove_fancy_hat(),
          _ => reroll(), // These values aren't that important
      }

      fn add_fancy_hat() {}
      fn remove_fancy_hat() {}
      fn reroll() {}
  ```

- If we want to tell Rust to literally do nothing, then we can use unit tuple `()` instead of fn call.

  ```rust
      let dice_roll = 9;
      match dice_roll {
          3 => add_fancy_hat(),
          7 => remove_fancy_hat(),
          _ => (), // Telling Rust to "do nothing"
      }

      fn add_fancy_hat() {}
      fn remove_fancy_hat() {}
  ```

#### The `if let` syntax

- It is used in case you want to consider only particular case of an enum.
- For example, if you want to consider only the `Some` variant of an enum `Option<>`, you may prefer to use the `if let` syntax instead of `match`:

  ```rust
  // The older approach using the match syntax
  let config_max = Some(3u8);
  match config_max {
      Some(max) => println!("The maximum is configured to be {}", max),
      _ => (), // This line seems redundant
  }

  // More concise approach with if let
  let config_max = Some(3u8);
  if let Some(max) = config_max {
      println!("The maximum is configured to be {}", max);
  }
  ```

- The `if let` accepts a pattern (consider `Some(max)`) and an expression (consider `config_max`) seperated by and `=` sign.
- Before using `if let` please make sure whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.
- This approach is not exhaustive in sense that it only considers one pattern and ignores other unlike the `match` syntax.

- It is possible to use `else` with `if let`:

```rust
// In this problem we are counting the coins that aren't quarter
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}

// Another possible approach with if let and else
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```
