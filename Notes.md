# Rust
- `const` can be used in the global scope, and `let` can only be used in a function
- Variables are immutable by default
- Shadowing: using `let` to perform a few transformations on a value but have the variable be immutable after those transformations have been completed
- Statically typed language so it must know the types of all variables at compile time
- Use `parse` to convert a type to another
  - Example: `let guess: u32 = "42".parse().expect("Not a number!");`
- Data types
  - Scalar types (4 in total)
    - Integers: e.g. `u32`
      - Signed numbers are stored using two’s complement (MSB == 1 ? negative : positive)
      - Signed variants can store numbers from -(2^n - 1) to 2^n - 1 - 1 inclusive, where n is the number of bits that variant uses. So an i8 can store numbers from -(2^7) to 2^7 - 1, which equals -128 to 127
      - `isize` and `usize` types depend on the computer architecture
        - Use when indexing collections
      - Number literals will ignore `_` visual separators
    - Floating-point numbers: `f32` and `f64` (roughly same speed as `f32` but has double precision)
    - `bool`
    - `char`: character literals use single quotes, string literals use double quotes
      - Four bytes to represent Unicode
  - Compound types
    - Tuples: fixed length and can hold any data type
      - Destructuring:
        ```rust
        let tup = (500, 6.4, 1);
        let (x, y, z) = tup;
        ```
      - Accessing elements: `tup.1`, `tup.2`, etc.
      - Unit tuple: tuple without any values that represent an empty value
    - Arrays: fixed length and single type
      - Init with elements: `let a: [i32; 5] = [1, 2, 3, 4, 5];`
      - Init repeated elements: `let a = [<val>; <n>]`;
      - Accessing elements: `a[0]`, `a[1]`, etc.
        - If the index is greater than or equal to the length, Rust will panic.
    - `panic!` causes the program to exit with an error
    - Integer overflow in release mode (`--release` flag) performs two’s complement wrapping
      - In development mode, it will panic at runtime
      - Can explicitly handle integer overflow with `wrapping_*`, `checked_*`, `overflowing_*`, and `saturating_*` methods
  - Functions:
    - Defining parameter types: `fn do_something(value: i32, bob: char)`
    - Declare function return type after an arrow
      - Example: `fn five() -> i32 { 5 }`
    - Last expression is returned implicitly
  - Rust is an expression-based language
    - Statements: perform an action, do not return a value, and must have ending semicolons
      - Assignments are statements (unlike in C, where assignments return the value of the assignment)
    - Expressions: evaluate to a resultant value and do not include ending semicolons
      - A new scope created with curly brackets `{}` are expressions:
        ```rust
        let y = {
          let x = 3;
          x + 1
        };
        ```
      - `if` is an expression, so it can be used on the right side of a `let` statement
        - If conditions must be a `bool`
      - Loops: `loop { println!("again!"); }`
        - Use `break` to break out of the innermost loop
        - Use `continue` to immediately move onto the next iteration of the innermost loop
        - Can be used on the right of a `let` statement, use `break` to return a value
          - Example:
            ```rust
            let mut counter = 0;
            let result = loop {
              counter += 1;
              if counter == 10 {
                break counter * 2;
              }
            };
            ```
        - Labels: `'counting_up: loop`
      - Conditional loops: `while <condition> { }`
      - Iterate over a collection: `for element in a { }` (this is the safest option)
      - Using a `Range`: `for number in (1...4) { }`
  - Ownership
    - Copying is the same as C. You just copy the pointer (“shallow copy”), which makes the first reference now invalid.
      - Example:
        ```rust
        let s1 = String::from("hello");
        let s2 = s1; // s1 is now invalid
        ```
    - When a variable goes out of scope, Rust automatically calls the `drop` function and cleans up the heap memory for that variable
    - Passing a variable to a function will move or copy just as assignment does
      - Example:
        ```rust
        let s = String::from("hello"); // s comes into scope
        takes_ownership(s); // s's value moves into the function...
        // ... and so is no longer valid here
        ```
    - Borrowing is the action of creating a reference
      - References are immutable by default
        - Creating a mutable reference and passing it into a function: change variable to `mut`, create a mutable reference with `&mut <var>`, pass that into the function which has a `&mut <vartype>` type parameter
      - Cannot have one or more immutable references and a mutable reference
      - Cannot have more than one mutable references (could cause data races)
      - Can have a mutable reference defined in another scope or defined after the immutable references have gone out of scope or are no longer being used
      - Dangling references are those that refer to a location in memory that has changed ownership (freeing some memory while preserving a pointer to that memory)
        - Rust ensures this never happens by returning an error
      - `clear()` requires a mutable reference, so it cannot be called if an immutable reference exists
      - `rect1.can_hold(&rect2)` passes in `&rect2`, which is an immutable borrow to `rect2`, an instance of `Rectangle`
    - Slices are immutable references that refer to a contiguous sequence of elements in a collection
      - Slices store a reference to the first element and a length
      - The ending index is exclusive
      - Example:
        ```rust
        let s = String::from("hello world");
        let hello = &s[0..5];
        let world = &s[6..11];
        ```
      - `&s[0..2]` is identical to `&s[..2]` as well as `&s[3..len]` and `&s[3..]`
      - `&s[..]` is a slice of the entire string
      - `&str` is the type that signifies a string slice
        - When you create a substring in Rust using methods like `split`, `slice`, or similar, the substring typically holds a reference to the original string's data.
      - String literals are slices
      - Define a function to take `&str` as a parameter instead of a reference to a `String`
        - Can convert a `String` to a reference using `.as_str()` method
      - Arrays:
        ```rust
        let a = [1, 2, 3, 4, 5];
        let slice = &a[1..3]; // has type `&[i32]`
        assert_eq!(slice, &[2, 3]);
        ```
      - `iter()` returns each element in a collection and `enumerate()` wraps the result of `iter()` as a tuple instead, the first element being the index and the second being a reference to the element
      - Byte literal: `b' '`
  - Structs
    - Create an instance of a struct by specifying concrete values for each of the fields containing key: value pairs
    - Use dot notation to access a value or change a value if the instance is mutable
    - Field init shorthand syntax:
      ```rust
      fn build_user(email: String, username: String) -> User {
        User {
          active: true,
          username,
          email,
          sign_in_count: 1,
        }
      }
      ```
    - Struct update syntax: use `..prevInstance` to suggest that the remaining fields have the same value as the previous instance
      - If any of the fields use values with the `Move` trait, then the previous instance will become invalid
    - Tuple structs are structs without names associated with their fields; they just have the types of the fields. Useful for when you want to create tuples with their own types
      ```rust
      struct Color(i32, i32, i32);
      fn main() {
        let black = Color(0, 0, 0);
      }
      ```
      - Can be destructured and accessed similar to a tuple
    - Each struct you define is its own type
    - Ownership of struct data:
      - `String` is an owned type, `&str` is a reference. For structs to store references to data owned by something else requires the use of lifetimes
    - Unit-like structs
      ```rust
      struct AlwaysEqual;
      fn main() {
        let subject = AlwaysEqual;
      }
      ```
  - Methods are functions that are defined within the context of a struct, enum, or a trait object, and their first parameter is always `self`, which represents the instance of the struct the method is being called on
    - Method syntax is just like dot notation (instance.method())
  - Use the keyword `impl` to start an implementation block for a struct
    - Use `&self` in parameters (shorthand for `self: &Self`) to signify we only want to read the data in the struct, not write to it. If we wanted to write to it, we would use the `mut` keyword.
    - The `&` indicates the method is borrowing the `self` instance (no `&` would transfer ownership)
    - All functions defined inside are associated functions
    - A function without `&self` as its first parameter is not a method since it doesn’t need an instance
      - Often used for constructors:
        ```rust
        impl Rectangle {
          fn square(size: u32) -> Self {
            Self {
              width: size,
              height: size,
            }
          }
        }
        ```
      - To call an associated function, use the `::` syntax
    - Structs can have multiple `impl` blocks
  - Rust has automatic referencing and dereferencing: given the receiver and name of the method, Rust can figure out definitively whether the method is reading (`&self`), mutating (`&mut self`), or consuming (`self`)
    - In C and C++, you would use `object→something()` or `(*object).something()` if `object` was a pointer, or `object.something()` if you were calling on the object directly
  - Enumerations, or enums are ways of saying a value is one of a possible set of values
    - To create an instance of an `enum`, for example `IpAddrKind`: `IpAddrKind::V4`
    - Rather than put an enum inside a struct, attach the data to an enum directly: `enum IpAddr { V4(String), V6(String), } let home = IpAddr::V4(String::from("127.0.0.1"));`
    - Each variant can also have different types and amounts of associated data, so rather than having a struct for each variant (which creates a different type for each one), use a single enum so then you can have a single type to pass into functions
    - Can also define methods on enums using `impl`
  - No `null` in Rust, but the enum `Option<T>` has a `None` variant to represent the concept of a value being absent (must use this to have a value that can possibly be null, and then you must create a handler for the case where it is null). This means we never have to check for null on, for instance, `i8`
    - `enum Option<T> { None, Some(T), }` (`T` is a generic type – can hold any data type)
    - You have to convert an `Option<T>` to a `T` before you can perform `T` operations with it
  - Use `match` statements for running code particular to each enum variant (and must do so for each one: exhaustive)
    - `match coin { Coin::Penny => { println!("Lucky penny!"); 1 } Coin::Nickel => 5, Coin::Dime => 10, Coin::Quarter => 25, }`
    - Common to match against an enum, bind a variable to the data inside, and then execute code based on it
    - Use keyword `other` that acts like an `else` statement
    - Use `_` for matching any value that you don’t want to use
  - `if let` lets you match one pattern while ignoring the rest:
    - `if let <SomeEnum>(<some value inside SomeEnum>)= <variable> { <some code that uses some value> }
    - Can use `else` to handle all other cases
  - Package, crates, and modules, oh my!
    - Binary crates are programs you can compile to an executable that you can run and have a `main` functions
    - Library crates are programs that define functionality intended to be shared with multiple projects
    - Crate root is the source file
    - A package is a bundle of one or more crates containing a `Cargo.toml` file describing how to build each crate
    - `pub` makes a module public
    - `use` creates shortcuts to items
    - Module tree is similar to file directories
  - Vectors
    - Store one type
    - Stored next to each other in memory
    - `vec!` macro creates a new vector using values you give it, inferring the data type
    - Implemented using generics `Vec<T>`
    - `let mut v = Vec::new(); v.push(5);`
    - Accessing: indexing or using `get` method
      - Indexing will panic if you access an element outside of bounds
      - `get` will return `None` (good for user input)
    - Using `&` and `[]` will give you a reference to the element at the index value
    - Can’t have an immutable reference to an element and a mutable reference to the vector
  - Strings are wrappers around vectors and thus share a lot of the same functionality
    - Can use string slices
  - Hash Maps: keys and values must be of the same type
    - `std::collections::HashMap`
    - Similar to vectors: `HashMap<K, V>`
    - `insert` to add a key-value pair
    - `get` to retrieve a value
    - Use `entry` to check if a value exists, inserting one if it doesn’t, and then returning a mutable reference
  - Error handling
    - `Result<T, E>` is the type returned from the `Result` and `Ok` is used to indicate that the function was able to successfully get the data
    - `Err` is used to indicate that the function failed and includes the reason
    - `unwrap` will return the `Ok` variant if it exists, otherwise it will panic
    - `expect` is a more flexible version of `unwrap` that allows us to set our own panic message
    - Propagate errors by using `?`
      - This only works in functions that have a return type of `Result`
    - Use `match` to handle multiple error conditions
    - You can use the `Result` type to allow for the propagation of errors, but you don’t have to use it to catch an error if you don’t want to
  - Generics: define functions, structs, enums, and methods with placeholders for data types
    - Must be declared after the `fn` keyword but before the function name
      - Example:
        ```rust
        fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
          let mut largest = list[0];
          for &item in list {
            if item > largest {
              largest = item;
            }
          }
          largest
        }
        ```
      - The `+` means we want a generic type that implements both `PartialOrd` and `Copy`
        - The reason for `Copy` is that if you are able to move the type by copying it, you don’t want the original value to no longer exist
  - Lifetimes: a way of giving references a name so that we can refer to them in function signatures
    - The names must be generic as well
    - Lifetime annotations are a way of telling Rust how generic lifetime parameters relate to the references the parameters are associated with
    - The syntax for lifetime annotations is either `&'a str` or `&'a String`
      - `'a` is the lifetime annotation
    - In function signatures, lifetimes are used to describe the relationships between the lifetimes of various parameters and the return value
    - Lifetimes do not change the lifetimes of any values passed in or returned
    - By using lifetimes, we specify that the references in the signature must be valid for at least the same amount of time as the `lifetime` specified in the return value
    - The main aim of lifetimes is to prevent dangling references, which cause a program to reference data that no longer exists
  - Testing
    - Tests are placed in the same file as the code they are testing
    - Use `#[cfg(test)]` to signal to Rust that the code following the attribute is a test
    - Create a new module called `tests`
    - You can use `#[test]` on the `fn` keyword or on the outermost `fn` keyword
    - Use `assert!` or `assert_eq!` to test the code
    - Run tests with the command `cargo test`
  - Documentation
    - Doc comments use `///`
    - Test code inside doc comments to ensure it is correct
    - You can have as many `#![doc = "something"]` lines as you want
    - Pass in the `--document-private-items` flag to include documentation for private items
  - The `trait` keyword allows us to define shared behavior in an abstract way (similar to interfaces in Java or C#)
    - A trait tells the Rust compiler about functionality a particular type has and can share with other types
    - Functions can have default implementations in traits
    - If there are multiple traits with the same method name, the compiler will be unable to determine which one you mean
    - If you want to force a method to be implemented, use the `;` syntax to provide no implementation and make it a rule that anything implementing the trait must provide its own implementation
    - If you want a method to use a default implementation, then you don’t need the `;`
  - Closures: anonymous functions defined in a block of code and assigned to a variable for later use
    - Capture values from the environment in which they are defined
    - `move` keyword can be used to force the closure to take ownership of the values it uses in the environment, and hence always make the closure take ownership of the values it uses
    - When you create a closure, Rust infers which trait to use based on the function signature and how it captures variables
    - Closure type is anonymous, so use `Fn`, `FnMut`, and `FnOnce` traits to specify if it will capture the environment by reference (`&T`), mutable reference (`&mut T`), or by value (`T`)
  - Iterators: allows going through each item in a collection
    - Use the `iter` method on a vector
    - If the type you are iterating through is known, you can use the `collect` method to convert it into a collection
    - `iter` for immutable references, `iter_mut` for mutable references
    - `into_iter` consumes the original collection and returns an iterator
    - Iterator adaptors are methods called on the `iter` and `iter_mut` methods that modify the iterators produced by them
    - `map`: takes a closure and produces a new iterator
    - `filter`: takes a closure that takes each item from the iterator and returns `true` or `false`. It produces a new iterator that only contains the items for which the closure returned `true`
  - Smart pointers
    - `Box<T>` is a type that allocates data on the heap and then points to that data. Boxes don’t have performance overhead, other than storing the data on the heap, but they don’t have many extra capabilities. They also don’t have a size known at compile time.
    - The `Deref` trait allows smart pointers to reference data like regular references do
    - A common trait to implement is `Drop`, which is called when an instance goes out of scope. The `Drop` trait allows you to customize the behavior that happens when an instance of the smart pointer goes out of scope.
      - Rust calls `drop` automatically when a value goes out of scope
      - Can also call `drop` manually by calling `std::mem::drop`
    - When a `Box<T>` instance goes out of scope, the `drop` function is called, and the `Box<T>` can deallocate the space on the heap
  - `Rc<T>`: a reference counting smart pointer. It keeps track of the number of references to a value which determines whether or not a value has any other references to it. If it does, then it will not deallocate the value. Once the last reference is gone, the value can be deallocated
    - Only for use in single-threaded scenarios
    - A weak reference allows you to have multiple owners of a `Rc<T>` value but without creating reference cycles (which would prevent the value from being deallocated)
  - `RefCell<T>`: a mutable smart pointer. Similar to `Box<T>`, `Rc<T>`, and `Ref<T>`, the `RefCell<T>` type is a smart pointer because it enables immutable or mutable borrows checked at runtime instead of compile time
  - Concurrency
    - The `Arc<T>` type is a thread-safe version of `Rc<T>` (atomic reference counting)
    - `Mutex<T>` provides interior mutability, wrapping around the value that we want to mutate and ensuring only one thread can access the data at a time
    - `Mutex` is short for mutual exclusion
      - Mutex is a simple implementation of a lock
    - Threads are similar to processes: they both are independent sequences of execution, both have their own memory space, and both can receive signals
    - In Rust, threads are used for concurrent programming
    - Join handles are useful for waiting for a thread to finish
  - Unsafe Rust
    - Instead of the borrow checker, we enforce memory safety with a system of ownership and a set of rules. But sometimes, these rules are too restrictive.
    - Use the `unsafe` keyword and then declare a block of `unsafe` code. Inside this block, you can do four things that are unsafe in Rust:
      - Dereference a raw pointer
      - Call an unsafe function or method
      - Access or modify a mutable static variable
      - Implement an unsafe trait
    - Safe abstractions that the borrow checker relies on
    - Smart pointers: `Box<T>`, `Rc<T>`, and `RefCell<T>` implement the `Deref` trait. In particular, `RefCell<T>` allows for mutable borrows checked at runtime instead of compile time. The `UnsafeCell<T>` type is the building block that enables `RefCell<T>` to bend Rust’s rules.
    - We have traits that allow us to create new safe abstractions that the borrow checker doesn’t know about, which we call unsafe traits. The `Send` and `Sync` traits are examples of unsafe traits, and we used them when we introduced concurrency in Chapter 16. These traits tell the Rust compiler to treat a type as if it’s threadsafe.
    - Use of the `unsafe` keyword is not a way of telling Rust that it’s okay to run this code. Instead, it’s a way of saying that you’ve read this code and that you, the programmer, have ensured that this code upholds Rust’s guarantees, even though the compiler can’t verify that.
    - Fear not! Unsafe code doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use `unsafe` code, you’ll still get the same guarantees Rust always gives you.
  - Why `unsafe`? It's generally good practice to use safe, higher-level abstractions provided by Rust. Unsafe Rust should be used when you need to:

      - Work with external code that doesn’t follow Rust’s ownership rules.
      - Implement abstractions that Rust’s type system doesn’t allow you to express.
      - Write some kind of low-level operations where you can ensure safety in a way that the compiler can’t. This is the riskiest case, because if you’re wrong, it might lead to undefined behavior that could cause crashes or data corruption.

      - Unsafe Rust is somewhat of a last resort, when you’ve tried all other options and still can’t get the behavior you need. Even then, it’s better to isolate as little code as possible in an `unsafe` block and to make that block as small as possible. This will limit the amount of unsafe code in your program, reducing the risk of bugs.

      - Rust is all about trade-offs and trying to prevent bugs at compile time rather than pushing them to runtime. It’s important to consider the reasons Rust has the patterns it does. If you can, use the safe abstractions provided by Rust, as they make your code easier to understand and less prone to bugs. If you do need to use unsafe code, make sure you have a good reason and encapsulate the unsafe code in a safe abstraction as much as possible.
