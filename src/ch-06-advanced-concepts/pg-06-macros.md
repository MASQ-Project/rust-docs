## Macros

- Rust code that writes more rust code are called _Macros_. This kind of programming is called metaprogramming.
- Here are the following things that you can only do with macros and not functions:

  - Macros can take variable number of parameters, unlike functions. You can call `println!("hello")` with one argument or `println!("hello {}", name)` with two arguments.
  - Macros are expanded before the compiler interprets the meaning of the code. For example, macros can implement a trait on a given type. Functions can't because it gets called at runtime and a trait needs to be implemented at compile time.

- Drawbacks of Macros:
  - It's hard to read, write and maintain.
  - You can define functions anywhere, but you need to bring the macros in scope before you can call them.

#### Declarative Macros

- They are the most widely used types of macros.
- Also referred to as "macros by example", “`macro_rules!` macros”, or just plain “macros”.
- They are similar to `match` statements, except they match on literal rust code, instead of some value.
- Here is a simple implementation of the `vec!` macro:

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

- Explanation:

  - `#[macro_export]` - You can't export the macro without this line. For using this macro, you'll have to bring the crate into scope where this macro is defined.
  - `macro_rules! name-of-macro` - Then we declare the macro with the `macro_rules!` along with the name of the macro without the exclamation mark. In our case, `vec`.
  - `( $( $x:expr ),* ) =>` - This is the match arm of the macro. In our case, the macro has only one match arm, if such an expression is passed to the macro which doesn't fall into it, it'll fail. Some complex macros will have multiple match arms.
    - `( ) =>` - A parantheses surrounds the whole pattern. It indicates that this is a match arm.
    - `$( )` - Anything inside this parantheses will capture values.
    - `$x: expr` - This matches any Rust expression and gives the expression the name `$x`.
    - `,` - It means that the literal `,` might appear after the code that matches the code in `$()`.
    - `*` - It means that the pattern matches zero or more of whatever precedes the `*`.
    - `$()*` - For every time the expression in `$()` gets matched, the code inside `$()*` will get called.

- In Action:

  - So, when we'll write `vec![1, 2, 3]` it will get compiled into, the following code:

    ```rust
    {
      let mut temp_vec = Vec::new();
      temp_vec.push(1);
      temp_vec.push(2);
      temp_vec.push(3);
      temp_vec
    }
    ```

Note: This vector that we created over here can take any number of arguments of any type. The implementation of `vec!` macro in standard library only accepts data of one type and it also has some extra code for preallocating memory for those types.

#### Procedural Macros

- They act more like functions and they are a type of procedure.
- They don't match against a pattern.
- They simply accept some code, operates on it and produces some new code.
- There are three kinds of procedural macros:

  - Custom Derive
  - Attribute Like
  - Function Like

##### Custom Derive Macros

- Using custom derive macros looks like this (it is used over structs or enums):

  ```rust
  #[derive(HelloMacro)]
  struct Pancakes;
  ```

- Defining proceudral macros looks like this:

  ```rust
  use proc_macro;

  #[some_attribute] // This attribute tells us which kind of procedural macro we are creating
  pub fn some_name(input: TokenStream) -> TokenStream { // TokenStream is a type imported from the crate `proc_macro`. It represents a sequence of tokens.
  }
  ```

- An Example of a _Custom Derive_ macro:

  - What we want? We want to print the name of the struct which tries to call the function `hello_macro()`.

    ```rust
    use hello_macro::HelloMacro; // A trait which has an associated function hello_macro()
    use hello_macro_derive::HelloMacro; // A macro that we can use

    #[derive(HelloMacro)]
    struct Pancakes;

    fn main() {
        Pancakes::hello_macro(); // This will print "Hello, Macro! My name is Pancakes!"
    }
    ```

  - Part 1: Defining Traits

    - First of all define the trait in different crate, created using `cargo new hello_macro --lib`:

      ```rust
      // File: src/lib.rs
      pub trait HelloMacro {
          fn hello_macro();
      }
      ```

    - Then implement the trait for every struct (without using macro, this is what it looks like):

      ```rust
      // File: src/main.rs
      use hello_macro::HelloMacro;

      struct Pancakes;

      impl HelloMacro for Pancakes {
          fn hello_macro() {
              println!("Hello, Macro! My name is Pancakes!"); // Programmer will have to implement this fn for each struct.
          }
      }

      fn main() {
          Pancakes::hello_macro();
      }
      ```

    - Additionally, we can’t yet provide the `hello_macro` function with default implementation that will print the name of the type the trait is implemented on: Rust doesn’t have reflection capabilities, so it can’t look up the type’s name at runtime.

  - Part 2: Implementing Procedural Macros

    - At the time of this writing, procedural macros need to be in their own crate. Eventually, this restriction might be lifted. So, first create a new crate using:

      ```zsh
      cargo new hello_macro_derive --lib
      ```

    - This trait will work in parallel with the trait defined above. Both the traits are tightly related. Hence we'll have to keep both the crates (`hello_macro` and `hello_macro_derive`) in one directory. In case someone wants to use the macro, they'll have to pull both the crates as dependencies.

    - So inside the `cargo.toml` file of the crate `hello_macro_derive`, add the following lines:

      ```toml
      [lib]
      proc-macro = true

      [dependencies]
      syn = "1.0"
      quote = "1.0"
      ```

    - Now, we can define the macro inside the `src/lib.rs` file of the crate `hello_macro_derive`. The code for most of the procedural macros will be same as the code block below:

      ```rust
      // Filename: src/lib.rs
      use proc_macro::TokenStream; // this crate, proc_macro comes with rust, it allows to read and manipulate rust code from our code
      use quote::quote; // Transforms DeriveInput -> Rust Code
      use syn; // Transforms Rust Code -> DeriveInput

      #[proc_macro_derive(HelloMacro)] // This line makes sure that whenever a user specifies #[derive(HelloMacro)] on a type, it calls the below fn
      pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
          // Construct a representation of Rust code as a syntax tree
          // that we can manipulate
          let ast = syn::parse(input).unwrap();

          // Build the trait implementation
          impl_hello_macro(&ast)
      }
      ```

    - Once the line `let ast = syn::parse(input).unwrap();` is executed (let's say for the `struct Pancakes {};`), it creates a `DeriveInput` struct, which looks like this:

      ```rust
      DeriveInput {
          // --snip--

          ident: Ident {
              ident: "Pancakes",
              span: #0 bytes(95..103)
          },
          data: Struct(
              DataStruct {
                  struct_token: Struct,
                  fields: Unit,
                  semi_token: Some(
                      Semi
                  )
              }
          )
      }
      ```

  - Now, we can convert the `DeriveInput` into `TokenStream` using the function `impl_hello_macro(&ast)`:

    ```rust
    fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
        let name = &ast.ident; // type is not &str but &syn::Ident
        let gen = quote! { // it turns Rust syntax tree data structures into tokens of source code
            impl HelloMacro for #name {
                fn hello_macro() {
                    println!("Hello, Macro! My name is {}!", stringify!(#name)); // quote! is used here to replace #name with the value in the variable name
                }
            }
        };

        gen.into() // quote! can't directly convert into TokenStream so we call into()
    }
    ```

  - The `stringify!` macro used here is built into Rust. It takes a Rust expression, such as `1 + 2`, and at compile time turns the expression into a string literal, such as `"1 + 2"`. This is different than `format!` or `println!`, macros which evaluate the expression and then turn the result into a String.

  - Now, we can create a crate named `pancakes` and then use our macro inside of it.

    ```zsh
    cargo new pancakes
    ```

  - The file structure should be like this:

    ```zsh
    .
    ├── hello_macro
    │   ├── hello_macro_derive
    │   │   └── ..
    │   └── ..
    └── pancakes
        └── ..
    ```

  - Then migrate the code as explained in the starting of this example to `src/main.rs`, and also add these lines to the dependencies:

    ```toml
    hello_macro = { path = "../hello_macro" }
    hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
    ```

##### Attribute Like Macros

- In Custom Derive Macros, the `derive` keyword is used and it generates some new code for the struct or enum.
- Instead of generating new code, Attribute like macros allow you to create new attributes.
- Unlike Custom Derive Macros, Attribute like macros are not limited to just structs or enums and can be applied to other items, such as functions.
- Here's an example of how it can be used on a function:

  ```rust
  #[route(GET, "/")]
  fn index() {
    ..
  }
  ```

- This `#[route]` attribute would be defined by the framework as a procedural macro. The signature of the macro definition function would look like this:

  ```rust
  #[proc_macro_attribute]
  pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    // attr: The GET, "/" will get stored in this argument
    // item: The code attached to above macro (fn index() {} in our case) will get stored in this argument
    ..
  }
  ```

- Other than that, attribute-like macros work the same way as custom derive macros: you create a crate with the `proc-macro` crate type and implement a function that generates the code you want!

##### Function Like Macros

- These macros look like function calls but with a `!`.
- They're more flexible than functions as they can accept variable number of arguments.
- In declarative macros (`macro_rules !` macro) uses match-like syntax, the Function Like Macros take a `TokenStream` parameter, similar to the other two procedural macros.
- Here's an example, here we want to parse SQL code:

  ```rust
  let sql = sql!(SELECT * FROM posts WHERE id=1);
  ```

- If we tried to build this macro with the `macro_rules !` macro, then match-like pattern would've made it hard to implement. With using `TokenStream` it is a bit easier to implement:

  ```rust
  #[proc_macro]
  pub fn sql(input: TokenStream) -> TokenStream {
    ..
  }
  ```

- It's implementation is closer to that of custom derive macros.
