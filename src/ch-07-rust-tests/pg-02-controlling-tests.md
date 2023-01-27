## Controlling Tests

- Command Line Options either go to:

  1. `cargo test` - Run this command for help, `cargo test --help`
  2. The resulting test binary - Run this command for help, `cargo test -- --help`.

- In simple terms, if you want to pass something into tests binary you'll have to seperate them with `--`.

#### Threading while running Tests

- By default, Rust runs tests in different threads.

- In case the test depends on each other. For example, tests perform read, write, and assert to a same file. If run concurrently, the file will get corrupt and the tests might fail.

- To run tests on a single thread, one by one we can do it by:

  ```zsh
  cargo test -- --test-threads=1
  ```

- This command will prevent parallelism, though the tests will take longer to run but it'll save us from the above problem.

#### Printing while running tests

- By default, when we run `cargo test`, the compiler never prints the statements.

- To display print statements we can run this command:

  ```zsh
  cargo test -- --show-output
  ```

#### Running Specific Tests

- To run specific tests either 1 or more, we can do that by specifying the name of test.

  - For One Test:

    ```zsh
    cargo test <name-of-test>
    ```

  - For more than one test (e.g. add_two, add_three, then substring add):

    ```zsh
    cargo test <substring-of-tests-name>
    ```

- When we run specific tests, the remaining tests will fall into the category of `filtered out`, you may see that in the logs.

- Also note that the module in which a test appears becomes part of the test’s name, so we can run all the tests in a module by filtering on the module’s name.

#### Ignoring Tests

- Instead of filtering tests, you may like to ignore the tests by adding `#[ignore]` attribute above each test that you'd like to ignore.

- Now, you may use the following commands to run the tests.

  - To run the tests ignoring ones that use `#[ignore]` attribute:

    ```zsh
    cargo test
    ```

  - To run only the ignored tests:

    ```zsh
    cargo test -- --ignored
    ```

  - To run both ignored and not ignored tests:

    ```zsh
    cargo test -- --include-ignored
    ```
