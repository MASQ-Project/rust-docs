
### Smart Pointers

- Differences between Pointer and Smart Pointer:

| Pointer                                                                           | Smart Pointer                                                                                                                               |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| A pointer is a general concept for a variable that contains an address in memory. | Smart pointers, on the other hand, are data structures that not only act like a pointer but also have additional metadata and capabilities. |
| References are pointers that only borrow data.                                    | Smart pointers _own_ the data they point to.                                                                                                |
| The most common kind of pointer in Rust is a reference and is indicated by `&`.   | The commonly used smart pointers are `String` and `Vec<T>`.                                                                                 |

- Smart pointers are usually implemented using `structs`.
- The characteristic that distinguishes a smart pointer from an ordinary struct is that smart pointers implement the `Deref` and `Drop` traits.
  - `Deref`: Allows an instance of the smart pointer struct to behave like a reference so you can write code that works with either references or smart pointers.
  - `Drop`: Allows you to customize the code that is run when an instance of the smart pointer goes out of scope.

#### `Box<T>`

- For allocating values on Heap.
- Box allows you to store _data on heap_ and the _pointer to the heap data on stack_.
- You’ll use them most often in these situations:
  - When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
  - When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
  - When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type
- Once Box goes out of scope, the deallocation happens for the box (stored on the stack) and the data it points to (stored on the heap).

- Using `Box<T>` for the recursive call:

  - Let's try to create an enum which will create it's variant recursively:

    ```rust
    // FAIL: While computing Size for Cons, Rust will detect an inifinite memory allocation
    enum List {
        Cons(i32, List),
        Nil,
    }

    use crate::List::{Cons, Nil};

    fn main() {
        let list = Cons(1, Cons(2, Cons(3, Nil)));
    }
    ```

  - When Rust will try to recognize the size, it'll recognize that it is an infinite loop:

    <img src="https://doc.rust-lang.org/book/img/trpl15-01.svg" alt="Infinite List Containing Infinite Cons Value" width="400"/>

  - Now, to make it easier for Rust to identify the size of enum at compile time, we can use `Box<T>` for the recursive call. Since `Box<T>` is a pointer, Rust will need not to find the size what's inside of it, instead just allocate the memory for it's pointer.

    ```rust
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }

    use crate::List::{Cons, Nil};

    fn main() {
        let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
    }
    ```

  - Conceptually, we still have a list, created with lists “holding” other lists, but this implementation is now more like placing the items next to one another rather than inside one another.

    <img src="https://doc.rust-lang.org/book/img/trpl15-02.svg" alt="List that is not infinitely sized" width="400"/>

- Since `Box<T>` implements the `Deref` trait, so you can use the `*` operator to dereference it.

#### `Deref` Trait

- Note that the `*` operator is replaced with a call to the deref method and then a call to the `*` operator. It means that `*y`, translates into:

  ```rust
  *(y.deref())
  ```

- We're trying to re-create `Box<T>` and it's capabilities to dereference itself. One important thing to notice, here we're only mimicing the dereferencing because the data doesn't actually get stored on heap.

  ```rust
  use std::ops::Deref;

  struct MyBox<T>(T);

  impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T>  {
        MyBox(x)
    }
  }

  impl<T> Deref for MyBox<T> {
      type Target = T;

      fn deref(&self) -> &Self::Target {
          &self.0
      }
  }
  ```

#### Deref Coercion

- Deref coercion can convert `&String` to `&str`. It's possible because `String` implements the `Deref` trait such that it returns `&str`.
- They are meant for the arguments of functions and methods. The ease is that, you can pass `&String` into a function that accepts `&str`:

  ```rust
  fn hello(name: &str) {
      println!("Hello, {}!", name);
  }

  fn main() {
    let name = String::from("Bob");
    hello(&name);
  }
  ```

- Deref coercion was added to Rust so that programmers writing function and method calls don’t need to add as many explicit references and dereferences with `&` and `*`.

- How does Rust automatically converts `&String` to `&str`?

  - It happens because `Deref` trait is implemented.
  - Rust simplifies all the deref implementations.

- Here's an even complex example of deref coercion, using the `MyBox`, that we created earlier:

  - Instead of calling this:

    ```rust
    fn main() {
        let m = MyBox::new(String::from("Rust"));
        hello(&(*m)[..]);
    }
    ```

  - Here we are manually converting:

    ```rust
    (*m) => MyBox<String> -> String
    [..] => String -> str
    & => str -> &str
    ```

  - We can just call this:

    ```rust
    fn main() {
        let m = MyBox::new(String::from("Rust"));
        hello(&m);
    }
    ```

  - Rust simplifies the deref implementations by calling the `deref()` again and again. First It'll call `deref()` for `MyBox` then for `String`.

    ```rust
    &MyBox<String> -> &String -> &str
    ```

Note: When the Deref trait is defined for the types involved, Rust will analyze the types and use `Deref::deref` as many times as necessary to get a reference to match the parameter’s type. The number of times that `Deref::deref` needs to be inserted is resolved at compile time, so there is no runtime penalty for taking advantage of deref coercion!

- Rust does deref coercion when it finds types and trait implementations in three cases:

  - From `&T` to `&U` when `T: Deref<Target=U>`
  - From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
  - From `&mut T` to `&U` when `T: Deref<Target=U>`

Note: The first case states that if you have a `&T`, and `T` implements `Deref` to some type `U`, you can get a `&U` transparently. the second case is similar. The third case is a bit different as mutable reference changes into immutable reference, though vice versa is not true.

#### `Drop` Trait

- In Rust, you can specify that a particular bit of code be run whenever a value goes out of scope, and the compiler will insert this code automatically.
- As a result, you don’t need to be careful about placing cleanup code everywhere in a program that an instance of a particular type is finished with—you still won’t leak resources!

  ```rust
  struct CustomSmartPointer {
      data: String,
  }

  impl Drop for CustomSmartPointer {
      fn drop(&mut self) {
          println!("Dropping CustomSmartPointer with data `{}`!", self.data);
      }
  }

  fn main() {
      let c = CustomSmartPointer {
          data: String::from("my stuff"),
      };
      let d = CustomSmartPointer {
          data: String::from("other stuff"),
      };
      println!("CustomSmartPointers created.");
  }
  // Output -
  // CustomSmartPointers created.
  // Dropping CustomSmartPointer with data `other stuff`!
  // Dropping CustomSmartPointer with data `my stuff`!
  ```

- Notice that, variables are dropped in the reverse order, `d` was dropped before `c`.
- There might be some cases when you want to manually drop the Smart Pointer, instead of waiting for the scope to end. For example, you want to release the lock so that other code in the same scope can acquire the lock.

  - First thing is that, you can't call the `drop()` from the `Drop` trait.

    ```rust
    // FAIL: This is not allowed in Rust, compiler will throw "explicit destructor calls not allowed"
    fn main() {
        let c = CustomSmartPointer {
            data: String::from("some data"),
        };
        println!("CustomSmartPointer created.");
        c.drop();
        println!("CustomSmartPointer dropped before the end of main.");
    }
    ```

    ```zsh
      --> src/main.rs:16:7
       |
    16 |     c.drop();
       |     --^^^^--
       |     | |
       |     | explicit destructor calls not allowed
       |     help: consider using `drop` function: `drop(c)`
    ```

  - Compiler uses term `destructor`, which is the general programming term for a function that cleans up an instance. It is analogous to the word `constructor`.
  - The reason that compiler doesn't allows us to do that, is to prevent the _double free error_.
  - The alternative is to use `drop()` from `std::mem::drop`, good thing is it's already in the `prelude`, so you don't need to import it.

    ```rust
    fn main() {
        let c = CustomSmartPointer {
            data: String::from("some data"),
        };
        println!("CustomSmartPointer created.");
        drop(c); // Notice, we're passing it as an argument
        println!("CustomSmartPointer dropped before the end of main.");
    }

    // Ouput -
    // CustomSmartPointer created.
    // Dropping CustomSmartPointer with data `some data`!
    // CustomSmartPointer dropped before the end of main.
    ```

  - It solves the _double free error_ through the ownership rules, as we pass it as an argument.

#### `Rc<T>`

- Also known as _Reference Counted Smart Pointer_, it allows multiple ownership.
- It does that by keeping the count of references. When the count becomes 0, it means that there are no references linked to data, so it's safe to clean.
- Imagine `Rc<T>` as a TV in a family room. When one person enters to watch TV, they turn it on. Others can come into the room and watch the TV. When the last person leaves the room, they turn off the TV because it’s no longer being used. If someone turns off the TV while others are still watching it, there would be uproar from the remaining TV watchers!
- Use case:

  - We want to allocate data on heap and we want multiple parts of our code to read it.
  - The problem is that we don't know which part will stop reading it last, that's why we can't make someone as an owner.
  - For those cases, `Rc<T>` will help us, it'll save us from deciding someone as owner and will allow multiple parts to read it at the same time.

- With `Rc<T>` it is possible to create two lists that both share ownership of a third list.

  <img src="https://doc.rust-lang.org/book/img/trpl15-03.svg" alt="List that is not infinitely sized" width="400"/>

  - Trying to do this with `Box<T>` fails:

    ```rust
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }

    use crate::List::{Cons, Nil};

    fn main() {
        let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
        let b = Cons(3, Box::new(a));
        let c = Cons(4, Box::new(a));
    }
    ```

  - We can also use references with lifetimes to solve this problem, but `Rc<T>` is better here.

    ```rust
    enum List {
        Cons(i32, Rc<List>),
        Nil,
    }

    use crate::List::{Cons, Nil};
    use std::rc::Rc;

    fn main() {
        let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
        let b = Cons(3, Rc::clone(&a)); // Notice that we don't need to use Rc<T> here, since no one will be owning b and c
        let c = Cons(4, Rc::clone(&a)); // Also notice that we're using Rc::clone() and passing reference to create owners
    }
    ```

  - `Rc::clone()` never makes deep copy, unlike `clone()`. `Rc::clone()` only increments the reference count, which doesn’t take much time.
  - To Increase Count: `Rc::clone()`, To Decrease Count: `Drop` does when the variable goes out of scope.

Note: `Rc<T>` can only allow multiple owners to read data and not to mutate it. For interior mutability there is another Smart Pointer named `RefCell<T>`.

#### `RefCell<T>`

- Allows interior mutability to the immutable data.
- Interior mutability is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data; normally, this action is disallowed by the borrowing rules.
- It uses `unsafe` rust code to function.

- An example:

- Consider one `trait` named `Messanger` and another `struct` named `LimitTracker`.

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
  messenger: &'a T,
  value: usize;
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
      ...
    }

    pub fn set_value(&mut self, value: usize) { // Problem 1: We want mutable reference of self, but it includes immutable reference to messenger
      self.value = value;

      if (value > 100) {
        self.messenger.send("Error: You are over your quota!"); // Problem 2: send() is an immutable function in trait, but self should be mutable.
      }
    }
}
```

- `LimitTracker` takes in a reference of `struct` that implements `Messenger`, so that it can store it as one of it's field.
- Inside `LimitTracker`, the problem is that `set_value()` takes **mutable reference** of `self`, but the `messenger` is an immutable reference and it's function send is also `immutable`. So, how can we test this `set_value()`?

- This will fail to compile:

  ```rust
  // FAIL: send() is required to be mutable by LimitTracker but immutable due to trait Messenger
  #[cfg(test)]
  mod tests {
      use super::*;

      struct MockMessenger {
          sent_messages: Vec<String>,
      }

      impl MockMessenger {
          fn new() -> MockMessenger {
              MockMessenger {
                  sent_messages: vec![], // This is immutable
              }
          }
      }

      impl Messenger for MockMessenger {
          fn send(&self, message: &str) {
               // We're trying to push on immutbale field, we also can't make send() mutable
              self.sent_messages.push(String::from(message));
          }
      }

      #[test]
      fn it_sends_an_over_75_percent_warning_message() {
          let mock_messenger = MockMessenger::new();
          let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

          limit_tracker.set_value(80);

          assert_eq!(mock_messenger.sent_messages.len(), 1);
      }
  }
  ```

- Here's the solution using `RefCell<T>`:

  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;
      use std::cell::RefCell;

      struct MockMessenger {
          // RefCell will make sent_messages mutable even though
          // it's parent MockMessenger can seem immutable to others
          sent_messages: RefCell<Vec<String>>,
      }

      impl MockMessenger {
          fn new() -> MockMessenger {
              MockMessenger {
                  // Now, RefCell will wrap the vector and will allow it to be mutable
                  // at places where it's parent is asked to be immutable
                  sent_messages: RefCell::new(vec![]),
              }
          }
      }

      impl Messenger for MockMessenger {
          fn send(&self, message: &str) {
              // MockMessenger will seem immutable to send() but
              // sent_messages is mutable, and items can be pushed into it
              self.sent_messages.borrow_mut().push(String::from(message)); // borrow_mut() will generate mutable reference for push()
          }
      }

      #[test]
      fn it_sends_an_over_75_percent_warning_message() {
          // --snip--

          assert_eq!(mock_messenger.sent_messages.borrow().len(), 1); // borrow() will generate the immutable reference, since we're only reading
      }
  }
  ```

- We use the `&` and `&mut` syntax with references. With `RefCell<T>`, we use the `borrow()` and `borrow_mut()` methods and they return `Ref<T>` and `RefMut<T>` respectively. They both implement `Deref` trait.

- `RefCell<T>` lets us have many immutable borrows or one mutable borrow at any point in time. It keeps a count of whenever we call the `borrow()`.
- In case of an error, it won't be just a compile error, but will appear on Runtime, and will cause a panic.

#### Differences between `Box<T>`, `Rc<T>` and `RefCell<T>`

| Property                 | `Box<T>`                        | `Rc<T>`                            | `RefCell<T>`                       |
| ------------------------ | ------------------------------- | ---------------------------------- | ---------------------------------- |
| Ownership                | Single Ownership                | Multiple Ownership                 | Single Ownership                   |
| Mutability of Inner Data | Immutable or Mutable            | Only Immutable                     | Immutable or Mutable               |
| Borrowing Rules Check    | Compiled Time (compiler errors) | Compiled Time (compiler errors)    | Runtime (panics at runtime)        |
| Multithreading           |                                 | Only for Single Threaded Scenarios | Only for Single Threaded Scenarios |

#### Using `RefCell<T>` with `Rc<T>`

- It'll give you superpowers.
- Now you can have a value that has multiple owners and can also mutate.
- To use it, you'll have to wrap it like this, `Rc<RefCell<T>>`.
- Here's our modified `Cons` list:

  ```rust
  enum List {
      Cons(Rc<RefCell<i32>>, Rc<List>),
      Nil,
  }
  ```

- Now, we can have a list like this:

  ```zsh
  b --|
      a---Nil
  c --|
  ```

- Here, `a` can have multiple owners `b` and `c`, and it's value can also mutate.

  ```rust
  fn main() {
      let value = Rc::new(RefCell::new(5)); // Created a value that can have multiple owners and can also mutate

      let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil))); // Make a such that it can be owned by multiple people

      let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a)); // Make b the owner of a
      let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a)); // Made c the owner of a

      *value.borrow_mut() += 10; // Rc -> RefCell -> &mut -> inner_element, then 10 is added to the inner_element in place

      println!("a after = {:?}", a); // Value of a: Cons(RefCell { value: 15 }, Nil)
      println!("b after = {:?}", b); // Value of b: Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
      println!("c after = {:?}", c); // Value of c: Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
  }
  ```

- Other Types to provide interior mutability:
  - `Cell<T>`: It copies the data instead of giving references.
  - `Mutex<T>`: It provides interior mutability that's safe to use in multiple threads.

#### Memory Leak

- When we accidentally create memory that is never cleaned up is called _Memory Leak_.
- Rust’s memory safety guarantees make it difficult, but not impossible.
- Rust allows memory leaks by using `Rc<T>` and `RefCell<T>`.
- By using both of them together, it's possible to create a _reference cycle_.
- A _reference cycle_ happens when reference of `a` is owned by `b` and reference of `b` is owned by `a`.
- First of all, this will cause an infinite loop of references.
- Also, it'll be impossible for Rust to `Drop` the values of `a` and `b`, as their reference count will never be zero.
- This is one of Rust's limitations, and is referred to the _problem of Memory Leak_.

- Reference Cycle in Action:

  ```rust
  use crate::List::{Cons, Nil};
  use std::cell::RefCell;
  use std::rc::Rc;

  #[derive(Debug)]
  enum List {
      Cons(i32, RefCell<Rc<List>>), // We can replace the list in place now
      Nil
  }

  impl List {
      fn tail(&self) -> Option<&RefCell<Rc<List>>> {
          match self {
              Cons(_, item) => Some(item),
              Nil => None,
          }
      }
  }

  fn main() {
      let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil)))); // a = (5, Nil)


      let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a)))); // b = (10, a)

      if let Some(link) = a.tail() {
          *link.borrow_mut() = b; // a = (5, b)
      }

      // At this point, Rc Count of a: 2, b: 2

      // This will try to print the lists infinitely
      // and then will crash with the stack overflow error
      println!("a next item = {:?}", a.tail());
  }

  // In case, we comment out the println!()
  // At the end of scope, Rust will try to decrease the count of references
  // Rc Count of b will decrease to 1
  // Rc Count of a will decrease to 1
  // Neither a nor b will be dropped as their count is not 0 and
  // will still stay on the heap, causing Memory Leak
  ```

- You may take help of this diagram to understand better:

  <img src="https://doc.rust-lang.org/book/img/trpl15-04.svg" alt="List that is not infinitely sized" width="400"/>

Important Lesson:
You can't rely on Rust's memory safety while using `RefCell<T>` and `Rc<T>` together, or another combination with interior mutability, as it may cause the problem of _Memory Leak_.

Solutions to prevent Reference Cycles:

- Use Automated tests, Code reviews, and other Software development practices to minimize.
- Reorganize data structure so that some references express ownership and some references don't.

| Strong Count                                               | Weak Count                                                                             |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Rust will only drop an element if this count becomes zero. | Rust will drop an element in case it gets out of scope, even if the count is not zero. |
| `Rc<T>` uses strong count.                                 | `Weak<T>` uses weak count.                                                             |
| It's hard to prevent reference cycle.                      | It's easier to prevent reference cycle.                                                |

- Preventing Reference Cycle using `Weak<T>`:

  ```rust
  use std::cell::RefCell;
  use std::rc::{Rc, Weak};

  #[derive(Debug)]
  struct Node {
      value: i32,
      parent: RefCell<Weak<Node>>, // Child won't own it's parent
      children: RefCell<Vec<Rc<Node>>>,
  }

  fn main() {
      let leaf = Rc::new(Node {
          value: 3,
          parent: RefCell::new(Weak::new()),
          children: RefCell::new(vec![])
      });

      let branch = Rc::new(Node {
          value: 5,
          parent: RefCell::new(Weak::new()),
          children: RefCell::new(vec![Rc::clone(&leaf)])
      });

      // The downgrade() changes Rc -> Weak
      *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

      // The upgrade() returns Some() or None, representing the value
      println!("Leaf parent: {:?}", leaf.parent.borrow().upgrade());
  }
  ```

- The output will look like this:

  ```zsh
  leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
  children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
  children: RefCell { value: [] } }] } })
  ```

- Notice, at some places only `Weak` is there, and the whole element is not printed. This is Rust's way of preventing infinite output.
- This lack of inifinite output, indicates that reference cycle is prevented.

- Here's another example, using the same struct to show how strong count and weak count will change:

  ```rust
  fn main() {
      let leaf = Rc::new(Node {
          value: 3,
          parent: RefCell::new(Weak::new()),
          children: RefCell::new(vec![]),
      });

      // leaf strong = 1, weak = 0

      {
          let branch = Rc::new(Node {
              value: 5,
              parent: RefCell::new(Weak::new()),
              children: RefCell::new(vec![Rc::clone(&leaf)]),
          });

          // Since Rc is downgrading branch, branch's weak count will increase by 1
          *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

          // branch strong = 1, weak = 1

          // leaf strong = 2, weak = 0
      }

      // leaf strong = 1, weak = 0
  }
  ```
