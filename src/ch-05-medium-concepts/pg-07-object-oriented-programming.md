## OOP (Object Oriented Programming)

- Characterstics of OOP:
  - _Objects contain data and behaviour_:
    - Programs should make up of objects.
    - An object packages both data and it's methods.
    - Rust offers `struct`, `enum` and `impl` blocks to provide this characterstic.
  - _Encapsulation_:
    - When the implementation details are hidden from the code that is using the object.
    - The only way to use an object is through it's public API.
    - This enables the programmer to change and refactor an object’s internals without needing to change the code that uses the object.
    - Rust encapsulates everything by default and offers `pub` keyword to make things public.
  - _Inheritance_:
    - When an object can inherit from another object’s definition, so that it can use it's parent object's data and behavior without defining them again.
    - Rust doesn't offers inheritance between `struct`, but has `trait` where you can use default methods that is both reusable and can be overridden.
    - The reasons to use inheritance are: _code reusability_ and _polymorphism_.
    - Polymorphism means that you don't need to explicitly define the type in code, but can be detected during runtime. It is useful when two types share same characterstics.
    - Rust offers polymorphism in a more general manner. It offers `generics` to generalize the accepted types and `trait bounds`, to constraint the allowed types.

Note: People think "polymorphism is synonymous with inheritance". But it is a more general concept which means that a certain code can be referred to multiple types. It is used when two types share some common characterstics. In inheritance, those types are only subclasses.

- Problems with Inheritance:

  - It adds the risk of sharing more code than necessary.
  - Subclasses are forced to share all the characterstics of the parents, even though sometimes it's not necessary or even undesired.
  - Sometimes calling the functions on the subclass doen't makes sense and even cause errors.
  - Due to this, some programming languages will only allow a subclass to inherit from one class, further restricting the flexibility of a program’s design.

- Rust is different, it takes a completely different approach by using trait objects instead of inheritance.

#### Defining a common behaviour using trait

- A `trait` object points to both:

  - An instance of a type implementing our specified trait
  - A table used to look up trait methods on that type at runtime.

| Property           | `struct` or `enum`          | `trait`                                      | Objects in other languages                 |
| ------------------ | --------------------------- | -------------------------------------------- | ------------------------------------------ |
| Stores Data        | Yes                         | No                                           | Yes                                        |
| Stores Behaviour   | No                          | Yes                                          | Yes                                        |
| Data and behaviour | Seperated by `impl` blocks. | Combined                                     | Combined                                   |
| Uses               | Store same items together   | Store common behaviour and allow abstraction | Store same items and their common behavior |

- An example:

  - _Problem:_ Let's say initially we have components such as `Button` and `Image` that may use a common functionality _to draw_ on the screen. It's possible that someday programmers want to introduce one more component named `SelectBox`. So, what we'll end up with are different types of structures that wants to use a common functionality.
  - _Solution:_ We can invent a common function named `draw()`, which will have different implementations for different types of components.
  - _How to build:_ We'll initialise a `trait` that can be shared among various components and a `struct` that can hold these compoenents.

  - The `trait` will look like this:

    ```rust
    // A common functionality shared between multiple components
    pub trait Draw {
        fn draw(&self);
    }
    ```

  - We can build a `struct` that holds the components that implements the `Draw`:

    ```rust
    pub struct Screen {
        // Box will allow to store the components on heap
        // dyn keyword will add the ability to detect a type that implements Draw on runtime
        pub components: Vec<Box<dyn Draw>>,
    }

    impl Screen {
        pub fn run(&self) {
            for component in self.components.iter() {
                component.draw();
            }
        }
    }
    ```

  - The difference with the alternative implementation using trait bounds in `struct` is that it restricts us to a `Screen` instance that has a list of components all of type `Button` or all of type `TextField`. At compile time, the definitions will be monomorphized.

    ```rust
    pub struct Screen<T: Draw> {
        pub components: Vec<T>,
    }

    impl<T> Screen<T>
    where
        T: Draw,
    {
        pub fn run(&self) {
            for component in self.components.iter() {
                component.draw();
            }
        }
    }
    ```

  - Programmers can now create new components like this:

    ```rust
    use gui::Draw;

    pub struct Button {
        // Some fields
    }

    impl Draw for Button {
        fn draw(&self) {
            // code to actually draw something
        }
    }
    ```

  - Users of this library can now use it like this:

    ```rust
    use gui::{Button, Screen};

    fn main() {
        let screen = Screen {
            components: vec![
                Box::new(SelectBox {
                    width: 75,
                    height: 10,
                    options: vec![
                        String::from("Yes"),
                        String::from("Maybe"),
                        String::from("No"),
                    ],
                }),
                Box::new(Button {
                    width: 50,
                    height: 10,
                    label: String::from("OK"),
                }),
            ],
        };

        screen.run();
    }
    ```

  - This concept is similar to the concept like _duck typing_: if it walks like a duck and quacks like a duck, then it must be a duck!

  - Use Cases:
    - Generics with trait bounds: If you’ll only ever have homogeneous collections. For Example, all elements of vector will be of type `Button`.
    - `Box<dyn T>`: You can use heterogeneous collections. For Example, elements can be a mix of `Button`, `TextField` etc.

- Static Dispatch vs Dynamic Dispatch:

| Static Dispatch                                                | Dynamic Dispatch                                                               |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Concrete types are decided at compile time.                    | The compiler emits code that at runtime will figure out which method to call.  |
| Compiler writes some new code for various concrete types.      | At runtime, it is decided whether a selected type can follow the requirements. |
| When we use trait bounds on generics, static dispatch happens. | When we want to perform dynamic dispatch, we can use the `dyn` keyword.        |
| No runtime cost is added.                                      | Some runtime cost is added.                                                    |

#### The State Pattern

- The _state pattern_ is an object-oriented design pattern.
- The current state is stored inside the struct, along with it's value(s).

  ```rust
  pub struct Post {
      // Box and dyn are used because the state variable
      // will have different states during the life of Post
      state: Option<Box<dyn State>>,
      content: String,
  }
  ```

- There are _state objects_, you can create a new object by implementing this trait.

  ```rust
  trait State {
      // The first two functions, results in transitions to different states.
      fn request_review(self: Box<Self>) -> Box<dyn State>;
      fn approve(self: Box<Self>) -> Box<dyn State>;
      // This function, can be called on any state object,
      // similar to the above two functions, except instead
      // of causing a state transition, it will return value
      // as if we conditionally returned output for each state
      fn content<'a>(&self, _post: &'a Post) -> &'a str {
          ""
      }
  }
  ```

  - The state pattern is built such that, methods defined on _state objects_ will cause changes in the `Post`, but the methods defined on `Post` will have no idea what these changes will look like. Hence, _state objects_ will encapsulate behaviour changes from the main _struct_.

  - You can look at it's complete implementation over [here](https://gist.github.com/utkarshg6/642859eef3d79fde55eeafb6cb4ed520).
  - Some downsides of State Pattern:

    - Extra Modifications: If we add a new state, we'll need to modify other states too. It's due to the reason that one state can only make transitions to another state.
    - Code Duplication: It leads us to write common code inside state objects, as we cannot write directly in trait's default implementation because traits don't know about the concrete type.

  - There's another implementation of state pattern in Rust. It doesn't follow the classic OOP pattern, as we'll require to store the object in new variable, whenever a state transition will happen. Here's the [code](https://gist.github.com/utkarshg6/507560be53345b20aca8304f477fa0b0).

    ```rust
    // This will cause a state transition from PendingReview to Published
    let post = post.approve();
    ```
