## Concurrency

- Difference between Concurrency and Parallel programming:

| Concurrent Programming                                  | Parallel Programming                                       |
| ------------------------------------------------------- | ---------------------------------------------------------- |
| Where different parts of program execute independently. | Where different parts of program execute at the same time. |

#### Using Threads

- OS manages multiple processes at once.
- An executed program's code is run in a process.
- You can write programs such that there are indpendent pieces of code that run simultaneously.
- The features that run these parts simultaneously are called _threads_.
- Threads can run simultaneously, there’s no inherent guarantee about the order in which parts of your code on different threads will run. This causes the following problems:
  - **Race conditions**: Where threads are accessing data or resources in an inconsistent order.
  - **Deadlocks**: Where two threads are waiting for each other to finish using a resource the other thread has, preventing both threads from continuing.
  - **Bugs**: Hard to reproduce bugs, and only happens in certain situations.
- Many operating systems provide an API for creating new threads. This model where a language calls the operating system APIs to create threads is sometimes called **_1:1_**, meaning _1 OS Thread / 1 Language Thread_. The rust standard library provides the implementation for only _1:1_.

- Creating a new thread with `spawn`:

  ```rust
  use std::thread;
  use std::time::Duration;

  fn main() {
    thread::spawn(|| {
      for i in 1..10 {
        println!("hi number {} from the spawned thread", i);
        thread::sleep(Duration::from_millis(1));
      }
    });

    for i in 1..5 {
      println!("hi number {} from the main thread", i);
      thread::sleep(Duration::from_millis(1));
    }
  }
  ```

- The output will be:

  ```zsh
  hi number 1 from the main thread
  hi number 1 from the spawned thread
  hi number 2 from the main thread
  hi number 2 from the spawned thread
  hi number 3 from the main thread
  hi number 3 from the spawned thread
  hi number 4 from the main thread
  hi number 4 from the spawned thread
  hi number 5 from the spawned thread
  ```

- The spawned thread will automatically die as the main thread ends.
- That's why spawned thread ran 5 times, 4 times same as main thread and the 5th time, which is exectued to break the condition for the main thread's for loop condition.
- Which thread will execute first is not guaranteed, you may notice in our case, the main thread runs first, even though according to the code, the spawned thread should have ran first.
- In fact, there is not even a guarantee that this spwaned thread will even run at all.
- We can make sure that the spawned thread will definitely run and will execute completely, by using the `join()`

  ```rust
  use std::thread;
  use std::time::Duration;

  fn main() {
      // Let's store the thread in a variable
      let handle = thread::spawn(|| {
          for i in 1..10 {
              println!("hi number {} from the spawned thread!", i);
              thread::sleep(Duration::from_millis(1));
          }
      });

      for i in 1..5 {
          println!("hi number {} from the main thread!", i);
          thread::sleep(Duration::from_millis(1));
      }

      // This will make sure that the spawned thread
      // finishes before the main thread ends
      handle.join().unwrap();
  }
  ```

- The two threads will now continue alternating, but the main thread will wait because of the call to `handle.join()` and will not end until the spawned thread is finished.
- It is very important, where you call the `handle.join()`, as it may create an unexpected behaviour:

  ```rust
  use std::thread;
  use std::time::Duration;

  fn main() {
      let handle = thread::spawn(|| {
          for i in 1..10 {
              println!("hi number {} from the spawned thread!", i);
              thread::sleep(Duration::from_millis(1));
          }
      });

      handle.join().unwrap();

      for i in 1..5 {
          println!("hi number {} from the main thread!", i);
          thread::sleep(Duration::from_millis(1));
      }
  }
  ```

- This will give us this output:

  ```zsh
  hi number 1 from the spawned thread!
  hi number 2 from the spawned thread!
  hi number 3 from the spawned thread!
  hi number 4 from the spawned thread!
  hi number 5 from the spawned thread!
  hi number 6 from the spawned thread!
  hi number 7 from the spawned thread!
  hi number 8 from the spawned thread!
  hi number 9 from the spawned thread!
  hi number 1 from the main thread!
  hi number 2 from the main thread!
  hi number 3 from the main thread!
  hi number 4 from the main thread!
  ```

- So, make sure that you're calling the `handle.join()` to prevent undesired behaviour.

- When we use closure, Rust will infer that we want to only borrow the variable.

  ```rust
  use std::thread;

  fn main() {
      let v = vec![1, 2, 3];

      // Notice, here v is only borrowed here,
      // it's possible that the closure may outlive
      // and v may die early, so Rust will throw us
      // error, and will ask us to use move
      let handle = thread::spawn(|| {
          println!("Here's a vector: {:?}", v);
      });

      handle.join().unwrap();
  }
  ```

- So, we need to expicitly add the `move` keyword, to tell Rust to transfer ownership of `v` to the closure.

  ```rust
  use std::thread;

  fn main() {
      let v = vec![1, 2, 3];

      let handle = thread::spawn(move || {
          println!("Here's a vector: {:?}", v);
      });

      // Now, we cannot use v over here, inside the main thread for any reason

      handle.join().unwrap();
  }
  ```

#### Using Message Passing to Transfer Data Between Threads

_“Do not communicate by sharing memory; instead, share memory by communicating.” - Go Language Documentation_

- Rust sends messages between threads to accomplish concurrency.
- Rust uses _channel_ for the message-sending concurrency (it works similar to a river stream), it has two parts:
  - _Transmitter_: The upstream location
  - _Receiver_: The downstream location
- A channel is said to be _closed_ if either the transmitter or receiver half is dropped.
- You may create a channel just like this:

  - A channel can have multiple producer of values (multiple sources of river), but only 1 consumer of those values (all rivers will mix into one river).
  - A channel produces it's two parts, inside a tuple and are abbreviated as `tx` and `rx`, for transmitter and receiver respectively.

    ```rust
    // mpsc stands for multiple producer, single consumer
    use std::sync::mpsc;
    use std::thread;

    fn main() {
        let (tx, rx) = mpsc::channel();

        thread::spawn(move || {
            let val = String::from("hi");
            // We'll send the value to the receiver's end
            // and in case there's a problem at receiving
            // end, it'll thrown an error and cause a panic
            tx.send(val).unwrap();
        });

        // The recv()
        let received = rx.recv().unwrap();
        println!("Got: {}", received);
    }
    ```

- Ways to receive the values from the channel:

  - `recv()`: It'll block the main thread’s execution and wait until a value is sent down the channel.
  - `try_recv()`: This method **doesn't block**, but may not contain any value for some time. So, you'll need to call this every so often, by writing a loop.

- Sending and Receiving multiple values:

  ```rust
  use std::sync::mpsc;
  use std::thread;
  use std::time::Duration;

  fn main() {
      let (tx, rx) = mpsc::channel();

      let tx1 = tx.clone();
      thread::spawn(move || {
          let vals = vec![
              String::from("hi"),
              String::from("from"),
              String::from("the"),
              String::from("thread"),
          ];

          for val in vals {
              tx1.send(val).unwrap();
              thread::sleep(Duration::from_secs(1));
          }
      });

      thread::spawn(move || {
          let vals = vec![
              String::from("more"),
              String::from("messages"),
              String::from("from"),
              String::from("you"),
          ];

          for val in vals {
              tx.send(val).unwrap();
              thread::sleep(Duration::from_secs(1));
          }
      });

      // We’re not calling the recv function explicitly anymore:
      // When the channel is closed, iteration will end.
      for received in rx {
          println!("Got: {}", received);
      }
  }
  ```

#### Shared-State Concurrency

- Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time.
- We can use _Mutex_ to allow access to data from one thread at a time.

##### Mutex

- _Mutex_ is an abbreviation of _Mutual Exclusion_.
- It locks the data such that others can use. Lock is a data structure that keeps track of who currently has exclusive access to the data.
- These are the rules that you'll have to follow while using a Mutex:
  - You must attempt to acquire the lock before using the data.
  - When you’re done with the data that the mutex guards, you must unlock the data so other threads can acquire the lock.
- Here's an example:

  ```rust
  use std::sync::Mutex;

  fn main() {
      let m = Mutex::new(5);

      {
          let mut num = m.lock() // It'll block the old thread, until we unlock the mutex
                         .unwrap(); // lock() may fail if the old thread panics, so unwrap() will also panic the current thread
          *num = 6; // Mutex returns a Smart Pointer named MutexGuard, that's why we need to dereference it to change it's value
      } // MutexGuard has a Drop trait implementation, which automatically unlocks the mutex when it goes out of scope

      println!("m = {:?}", m);
  }
  ```

- Here's an example wehere the varaible `counter` will be shared among 10 threads, where each of them will try to increment it by 1.

  - Why can't we directly use `Mutex` within multiple threads?
    - The threads use `move`, which moves the ownership of variable to the thread.
    - Rust won't allow us to move the ownership of lock counter in multiple threads.
  - Why can't we use `Rc<T>`, to provide multiple ownership to individual threads?
    - `Rc<T>` is not safe to share across threads. It is possible if we use `Rc<T>` in multiple threads, then both threads might update the reference count at same time.
    - It doesn’t use any concurrency primitives to make sure that changes to the count can’t be interrupted by another thread.
    - That could lead to Wrong Counts and Memory Leak.
  - What do we need then?
    - What we need is a type exactly like `Rc<T>` but one that makes changes to the reference count in a thread-safe way.
    - Fortunately, we have `Arc<T>` (atomically referenced counter), it is almost like `Rc<T>`, except the counts are maintained atomically.
    - Atomics are primitives that are safe to share across threads.

  ```rust
  use std::sync::{Arc, Mutex};
  use std::thread;

  fn main() {
      // Mutex is used to lock a variable so that other thread can use
      // Arc provides multiple ownership like Rc<T> and it is thread safe
      let counter = Arc::new(Mutex::new(0)); // Notice counter is immutable, it's because Mutex provides interior mutability, similar to RefCell
      let mut handles = vec![];

      for _ in 1..=10 {
          let counter = Arc::clone(&counter);
          let handle = thread::spawn(move || {
              let mut value = counter.lock().unwrap();
              *value += 1
          });

          handles.push(handle);
      }

      for handle in handles {
          handle.join().unwrap();
      }

      println!("The value of counter: {}", *counter.lock().unwrap());
  }
  ```

- The combination of `Arc<Mutex<T>>`, is analogous to `Rc<RefCell<T>>`.
- Keep in mind using `Mutex<T>` is risky, as logical errors may lead to _deadlocks_.

#### `Send` and `Sync` trait

- These two traits are part of the language itself, unlike otheer features of concurrency as they were part of the standard library.
- They are called `std::marker` traits.
- `Send` vs `Sync`

  | `Send`                                                               | `Sync`                                                                                   |
  | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
  | It is safe to transfer ownership of a type between threads.          | It is safe to use that type's reference betweeen threads.                                |
  | Any type `T` that implements `Sync`                                  | Type `T` is `Sync`, if it's reference (`&T`) is `Send` or if type `T` implements `Sync`. |
  | Except `Rc<T>`, almost all types are `Send`. (use `Arc<T>` instead). | Primitive Types, `Mutex<T>` are `Sync` but `Rc<T>`, `RefCell<T>` are not.                |

- We don't need to implement `Send` and `Sync` for the types that are made up of those types that implements these traits.
- In case you need to implement thes traits for a particular type than it means you'll need to write some `unsafe` rust code. You can learn the Dark Arts of Unsafe Code from this book [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/index.html).
