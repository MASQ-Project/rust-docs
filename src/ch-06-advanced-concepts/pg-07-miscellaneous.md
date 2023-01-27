## Miscellaneous

#### Raw Identifers

- Identifiers are the names of the functions, variable, structs etc.
- Keywords are the words reserved by Rust. For example, `if`, `else`, `match`.
- You're not allowed to use keywords as identifiers, but there's a hack.
- Raw identifiers are the syntax that lets you use keywords where they wouldn’t normally be allowed. You use a raw identifier by prefixing a keyword with `r#`.

- Here's something that you're not allowed to do:

```rust
// FAIL: match is a keyword, but is used as an identifier
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

- If there's a need to use a keyword as identifier, you may use raw identifiers:

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

- What's the use?
  - Let's say you're integrating some code written in other languages to your Rust project. Now, there are some functions written in other languages whose name are keywords in Rust. So, you may use raw identifiers to overcome Rust's restriction.
  - `try` isn’t a keyword in the 2015 edition but is in the 2018 edition. If you depend on a library that’s written using the 2015 edition and has a `try` function, you’ll need to use the raw identifier syntax, `r#try` in this case, to call that function from your 2018 edition code.

#### Derivable Traits

##### `Debug`

- It can used to print elements by using `:?` within `{}` placeholders.
- The `Debug` trait is required, for example, in use of the `assert_eq!` macro. This macro prints the values of instances given as arguments if the equality assertion fails so programmers can see why the two instances weren’t equal.

##### `PartialEq`

- By deriving this trait you can use `==` and `!=` operators on structs an enums.
- It implements the `eq` method.

##### `Eq`

- It's purpose is to check that the annotated types are equal to itself.
- It can only be implemented by the types that already implements `PartialEq`.
- The mathematical difference between `Eq` and `PartialEq` is that `Eq` is reflexive, symmetric and transitive. However,`PartialEq` only guarantees `symmetric` and `transitive`.

  ```math
  reflexive: a == a;
  symmetric: a == b implies b == a; and
  transitive: a == b and b == c implies a == c.
  ```

- An example of a type that follows `PartialEq` and doesn't follows `Eq` is floating point number. The implementation of floating point numbers states that two instances of the not-a-number (`NaN`) value are not equal to each other.
- An example of when `Eq` is required is for keys in a `HashMap<K, V>` so the `HashMap<K, V>` can tell whether two keys are the same.

##### `PartialOrd`

- A type that implements PartialOrd can be used with the `<`, `>`, `<=`, and `>=` operators.
- The `PartialOrd` trait allows you to compare instances of a type for sorting purposes.
- It implements the `partial_cmp` method, which returns an `Option<Ordering>` that will be `None` when the values given don’t produce an ordering.
- Since floating point numbers can't compare (or order) the not-a-number (`NaN`), it doesn't support this trait.
- When derived on enums, variants declared earlier in the enum definition are considered less than the variants listed later.
- When derived on structs, it compares two instances by comparing the value in each field in the order in which the fields appear in the struct definition.

##### `Ord`

- It implements the `cmp` method, which returns an `Ordering` rather than an `Option<Ordering>` because a valid ordering will always be possible.
- You can only apply the `Ord` trait to types that also implement `PartialOrd` and `Eq` (and `Eq` requires `PartialEq`).
- When derived on structs and enums, `cmp` behaves the same way as the derived implementation for `partial_cmp` does with `PartialOrd`.
- An example of when Ord is required is when storing values in a `BTreeSet<T>`, a data structure that stores data based on the sort order of the values.

##### `Clone`

- It allows you to perform a deep copy (copying the data on heap) of a value.
- When you derive clone on a type, then it is required that the type's fields must also derive the clone.

##### `Copy`

- It allows you to copy data on the stack.
- Copying is a lot faster than cloning but it's undesirable for large items. For Example, String.
- A type that implements copy must also implement clone. It is trivial because Clone performs the same task as Copy.

##### `Hash`

- It allows you to take an _instance of a type_ of arbitrary size and map that instance to a _value of fixed size_ using a hash function.
- It is required in storing keys in a `HashMap<K, V>` to store data efficiently.
- If a type implements this trait then it's required that all of it fields must also implement this trait.

##### `Default`

- It allows you to create a default value of a type.
- The `Default::default` function is commonly used in combination with the struct update syntax.
- You can customize a few fields of a struct and then set and use a default value for the rest of the fields by using `..Default::default()`.
- You can use the method `unwrap_or_default()` on `Option<T>` instances.
- For Example, If the `Option<T>` is `None`, the method `unwrap_or_default` will return the result of `Default::default` for the type `T` stored in the `Option<T>`.
