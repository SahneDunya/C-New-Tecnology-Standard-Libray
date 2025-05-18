# Using the CNT Standard Library
This document explains how to use the C New Technology (CNT) programming language's Standard Library (Stdlib) in your own code. The Stdlib provides a collection of pre-written modules and functions for performing common tasks.

1. What is the Standard Library and How is it Organized?
The CNT Standard Library is a collection of built-in modules providing essential functionalities such as fundamental data structures, I/O operations, file system access, networking, time management, and more. The Stdlib is organized into various sub-modules under the std root module:

std::core: Contains the language's most fundamental types (Option, Result, Ordering, basic numbers), traits (Copy, Drop), and runtime functions (panic). It typically provides essential building blocks.
std::io: Input/output operations for console and files (print, println, read_line).
std::collections: Common data structures (Vec, Map, Set).
std::math: Mathematical functions and constants (PI, sqrt, sin).
std::fs: File system operations (creating/deleting files/directories, reading/writing, metadata).
std::thread: Thread management facilities (spawn, join, sleep).
std::time: Time points and durations (Duration, SystemTime).
This modular structure of the Stdlib allows you to use only the parts you need in your code.

2. Including Standard Library Items in Code (import)
To use public (pub) items from another module (including Stdlib modules) in your code, you use the import statement.

The most common way is to import a specific module and access its items with a qualified name (module_name::Item):
```
// main.cnt
import std::io; // Import the std::io module

fn main() {
    // Use the println function from the std::io module
    io::println("Hello, Standard Library!");
}
```
If you want to give the imported module a shorter name (an alias), you can use the as keyword:
```
// main.cnt
import std::io as console; // Import the std::io module as 'console'

fn main() {
    // Use the println function via the 'console' alias
    console::println("Hello, Standard Library!");
}
```
You can also access sub-modules of imported modules using a chain:
```
import std::collections as coll; // Import the std::collections module
// To use the Vec type within the collections module:
let mut my_vec: coll::Vec<int>; // Assuming Vec supports generics and holds int
```
In some languages, import can also be used to bring specific items directly into the current scope (e.g., import std::io::println; -> println(...)). However, this can lead to name collisions. Using the module name or alias for access (module_name::Item) is a safe and clear method.

3. Using the Core Module (std::core)
The std::core module contains the most fundamental building blocks of the language. It is often automatically imported or widely used by other Stdlib modules. Key elements you should know about include:

Option<T> Enum: Used to represent situations where a value might not be present (similar to null but safer). It has two variants: core::Option::Some(value) (the value is present) and core::Option::None (the value is absent).
```
import std::core;
import std::io; // For printing

fn example_option(control: bool) -> core::Option<int> {
    if control {
        core::Option::Some(123); // Return an int value
    } else {
        core::Option::None; // Return no value
    }
}

fn main() {
    let result1 = example_option(true);
    let result2 = example_option(false);

    // Safely process the Option value with match
    match result1 {
        core::Option::Some(value) => io::println("Result1 contains a value: " + value.toString()), // toString assumed
        core::Option::None => io::println("Result1 is empty."),
    }

    match result2 {
        core::Option::Some(value) => io::println("Result2 contains a value: " + value.toString()),
        core::Option::None => io::println("Result2 is empty."),
    }

    // Warning: Using unwrap() can panic if the value is None!
    // let value = result2.unwrap(); // Error! Panics.
}
```
Result<T, E> Enum: Used to represent either a successful result (Ok) or an error (Err). It is common for operations that can fail (file operations, networking, time, parsing, etc.). It has two variants: core::Result::Ok(value) (the successful result value) and core::Result::Err(error) (the error value).
```
import std::core;
import std::fs; // File system operations return Result
import std::io;  // For printing

// Assuming a function like fs::read_to_string returns Result<string, fs::Error>

fn main() {
    let file_path = "test.txt";
    let read_result = fs::read_to_string(file_path); // fs::read_to_string returns Result

    // Safely process the Result value with match
    match read_result {
        core::Result::Ok(content) => {
            io::println("File read successfully:");
            io::println(content);
        },
        core::Result::Err(error) => {
            // Get the Error enum value
            io::print("File read error: ");
            // Need a mechanism to print the error enum (e.g., Debug/Display trait)
            io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

    // Warning: Using unwrap() can panic if the Result is Err!
    // let content = fs::read_to_string("non_existent_file.txt").unwrap(); // Error! Panics.
}
```
Ordering Enum: Represents the outcome of a comparison (core::Ordering::Less, core::Ordering::Equal, core::Ordering::Greater). Used with methods like cmp or in sorting algorithms.

panic!(message) Function: Immediately terminates the program or thread due to an unrecoverable error. Methods like unwrap() typically call panic! in case of an error or absence of value. Should be used cautiously; Result is preferred for controlled error handling.

4. Using Input/Output (std::io)
The std::io module provides functions for console and file I/O operations.
```
import std::io; // Import the std::io module

fn main() {
    // Prints a string to standard output without a newline
    io::print("Please enter your age: ");

    // Prints a string to standard output and appends a newline
    io::println("Hello!");

    // Reads a line from standard input and returns a string
    let input_text = io::read_line();

    io::println("You entered: " + input_text); // Assuming string concatenation is supported
}
```
5. Using Collections (std::collections)
The std::collections module provides common data structures like Vec (vector - growable array). Remember: This module requires support for generics (<T>), and your compiler needs to implement this feature. Examples below use the standalone function style.
```
import std::collections; // Import the collections module
import std::core;       // Import the core module for Option and Result
import std::io;         // For printing

fn main() {
    // Create a new, empty vector of integers
    // Requires generic syntax Vec<int>
    let mut numbers: collections::Vec<int> = collections::vec_new();

    // Append elements to the vector (vec_push takes mutable reference, moves value)
    collections::vec_push(&mut numbers, 10);
    collections::vec_push(&mut numbers, 20);
    collections::vec_push(&mut numbers, 30);

    // Get the length of the vector (vec_len takes immutable reference)
    let length = collections::vec_len(&numbers);
    io::println("Vector length: " + length.toString()); // toString assumed

    // Get an immutable reference to the element at a specific index (vec_get returns Option<&T>)
    let first_element_ref = collections::vec_get(&numbers, 0);
    match first_element_ref {
        core::Option::Some(value_ref) => io::println("First element: " + (*value_ref).toString()), // Dereference the reference
        core::Option::None => io::println("Vector is empty or index out of bounds."),
    }

    // Get a mutable reference to the element at a specific index (vec_get_mut returns Option<&mut T>)
    let mut second_element_mut_ref = collections::vec_get_mut(&mut numbers, 1);
     match second_element_mut_ref {
        core::Option::Some(value_mut_ref) => {
             *value_mut_ref = 25; // Modify the value through the mutable reference
             io::println("Second element updated.");
        },
        core::Option::None => io::println("Second element not found."),
    }

    // Remove the last element from the vector (vec_pop returns Option<T>, moves value out)
    let last_element = collections::vec_pop(&mut numbers);
    match last_element {
        core::Option::Some(value) => io::println("Popped last element: " + value.toString()), // Use the moved-out value
        core::Option::None => io::println("Vector was empty, no element popped."),
    }

     let current_length = collections::vec_len(&numbers);
    io::println("Vector length after pop: " + current_length.toString());

    // Note: The vector automatically cleans up its owned elements when Drop is required.
    // The vector itself is also Dropped when it goes out of scope.
}
```
6. Using Math (std::math)
The std::math module provides common mathematical constants and functions. Note that functions often have type-specific versions (_int, _float).

```
import std::math; // Import the math module
import std::io;   // For printing

fn main() {
    // Use constants
    let radius: float = 3.0;
    let area = math::PI * math::pow(radius, 2.0); // math::pow(float, float) -> float

    io::println("Area of circle (r=" + radius.toString() + "): " + area.toString()); // toString assumed

    // Use functions
    let number: float = 16.0;
    let square_root = math::sqrt(number); // math::sqrt(float) -> float
    io::println("Square root of " + number.toString() + ": " + square_root.toString());

    let angle: float = math::PI / 4.0; // 45 degrees
    let sin_value = math::sin(angle); // math::sin(float) -> float
    io::println("sin(" + angle.toString() + " radians): " + sin_value.toString());

    let integer_number: int = -50;
    let absolute_integer = math::abs_int(integer_number); // math::abs_int(int) -> int
    io::println("Absolute value of " + integer_number.toString() + ": " + absolute_integer.toString());

    let float_number: float = -15.7;
    let absolute_float = math::abs_float(float_number); // math::abs_float(float) -> float
    io::println("Absolute value of " + float_number.toString() + ": " + absolute_float.toString());
}
```
7. Using the File System (std::fs)
The std::fs module provides functions for interacting with files and directories. Important: Most functions in this module can fail and return results within core::Result. You must handle error cases.

Kod snippet'i
```
import std::fs;   // Import the file system module
import std::io;   // For printing
import std::core; // For Result enum variants and void

fn main() {
    let file_path = "example.txt";
    let dir_path = "new_directory";

    // --- Writing to a File (returns Result) ---
    let write_result = fs::write_string(file_path, "This is text written to a file with CNT!\nSecond line.");

    // Handle the Result with match
    match write_result {
        core::Result::Ok(void_value) => { // On success, it returns void (ignorable)
            io::println("File '" + file_path + "' written successfully.");
        },
        core::Result::Err(error) => {
            // On error, get the Error enum value
            io::print("Error! Could not write to file '" + file_path + "': ");
            io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

    // --- Reading from a File (returns Result<string, fs::Error>) ---
    let read_result = fs::read_to_string(file_path);

    match read_result {
        core::Result::Ok(content) => {
            io::println("Content of file '" + file_path + "':");
            io::println(content);
        },
        core::Result::Err(error) => {
            io::print("Error! Could not read file '" + file_path + "': ");
            io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

    // --- File/Directory Existence Check (Simple bool functions) ---
    let file_exists = fs::exists(file_path);
    io::println("File '" + file_path + "' exists: " + file_exists.toString()); // Assuming bool to string conversion

    let is_directory = fs::is_dir(file_path);
    io::println("'" + file_path + "' is a directory: " + is_directory.toString());


    // --- Creating a Directory (returns Result) ---
    let create_dir_result = fs::create_dir(dir_path);
    match create_dir_result {
        core::Result::Ok(void_value) => io::println("Directory '" + dir_path + "' created successfully."),
        core::Result::Err(error) => {
             io::print("Error! Could not create directory '" + dir_path + "': ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

     // --- Removing a Directory (returns Result - the directory must be empty) ---
    let remove_dir_result = fs::remove_dir(dir_path);
    match remove_dir_result {
        core::Result::Ok(void_value) => io::println("Directory '" + dir_path + "' removed successfully."),
        core::Result::Err(error) => {
             io::print("Error! Could not remove directory '" + dir_path + "': ");
             io::println(error.toString()); // Assuming the Error enum has a toString method (e.g., DirectoryNotEmpty error)
        }
    }


    // --- Removing a File (returns Result) ---
    let remove_file_result = fs::remove_file(file_path);
     match remove_file_result {
        core::Result::Ok(void_value) => io::println("File '" + file_path + "' removed successfully."),
        core::Result::Err(error) => {
             io::print("Error! Could not remove file '" + file_path + "': ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

     let file_exists_after = fs::exists(file_path);
    io::println("File '" + file_path + "' exists (after removal): " + file_exists_after.toString());
}
```
8. Using Threads (std::thread)
The std::thread module allows you to create and manage new threads. Remember: The thread::spawn function takes a function or closure to run. Your compiler needs to support this capability. Thread operations can also return errors (Result based).
```
import std::thread; // Import the thread module
import std::io;    // For printing
import std::core;  // For Result enum variants and void
import std::time;  // Used by sleep for Duration

// Function to be run in a new thread
// Its signature must match fn() -> core::void as expected by thread::spawn.
fn new_thread_task() -> core::void {
    io::println("Hello, speaking from a new thread!");
    // Note: Safe access/modification of shared data across threads requires synchronization primitives (Mutex, Channel)
}

fn main() {
    io::println("Main thread starting.");

    // Spawn a new thread (thread::spawn returns Result<thread::Thread, thread::Error>)
    let spawn_result = thread::spawn(new_thread_task); // Pass the function

    match spawn_result {
        core::Result::Ok(new_thread_handle) => {
            io::println("New thread spawned successfully.");
            io::println("Main thread waiting for the new thread to finish (joining)...");

            // Wait for the thread to finish (thread::thread_join moves the handle, returns Result<void, thread::Error>)
            let join_result = thread::thread_join(new_thread_handle); // The handle is moved

            match join_result {
                core::Result::Ok(void_value) => io::println("New thread finished and joined successfully."),
                core::Result::Err(error) => {
                    io::print("Error! Could not join thread: ");
                    io::println(error.toString()); // Assuming the Error enum has a toString method
                }
            }
        },
        core::Result::Err(error) => {
             io::print("Error! Could not spawn thread: ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

    // Sleeping the main thread for a duration (thread::sleep takes Duration, returns Result<void, time::Error>)
    io::println("Main thread sleeping for 1 second...");
    let one_second = time::duration_from_secs(1); // Use time::Duration
    let sleep_result = thread::sleep(one_second);

     match sleep_result {
        core::Result::Ok(void_value) => io::println("Main thread finished sleeping."),
         core::Result::Err(error) => {
             io::print("Error! Interrupted during sleep: ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
         }
    }

    io::println("Main thread ending.");
    // Note: When the main thread finishes, other detached threads may also terminate abruptly.
}
```
9. Using Time (std::time)
The std::time module allows you to manage time points (SystemTime) and durations (Duration).
```
import std::time; // Import the time module
import std::io;   // For printing
import std::core; // For Result enum variants, u64, etc.

fn main() {
    // --- Creating and Using Durations ---
    // Use functions like duration_from_secs, duration_from_millis, etc.

    let five_seconds = time::duration_from_secs(5);
    let fifty_milliseconds = time::duration_from_millis(50);
    let one_hundred_nanoseconds = time::duration_from_nanos(100);

    io::println("Five seconds in total nanoseconds: " + time::duration_as_nanos(&five_seconds).toString()); // as_nanos doesn't return Result
    io::println("Five seconds in total seconds: " + time::duration_as_secs(&five_seconds).toString());

    // Adding durations (checked_add returns Result<Duration, time::Error>)
    let total_duration_result = time::duration_checked_add(&five_seconds, &fifty_milliseconds);

    match total_duration_result {
        core::Result::Ok(total_duration) => {
             io::println("Total duration seconds: " + time::duration_as_secs(&total_duration).toString());
             io::println("Total duration nanoseconds (fractional): " + time::duration_subsec_nanos(&total_duration).toString());
        },
        core::Result::Err(error) => {
             io::print("Error! Duration addition: ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }


    // --- Using SystemTime ---
     system_time_now() returns Result<SystemTime, time::Error>

    let now_result = time::system_time_now();

    match now_result {
        core::Result::Ok(now) => {
            io::println("Current system time obtained.");
            // Calculate duration since epoch (returns Result<Duration, time::Error>)
            let dur_since_epoch_result = time::system_time_duration_since_epoch(&now);
            match dur_since_epoch_result {
                core::Result::Ok(epoch_duration) => {
                    io::println("Duration since epoch (seconds): " + time::duration_as_secs(&epoch_duration).toString());
                },
                core::Result::Err(error) => {
                     io::print("Error! Could not calculate duration since epoch: ");
                     io::println(error.toString()); // Assuming the Error enum has a toString method
                }
            }

            // Adding/subtracting a Duration (checked_add/sub return Result)
             let one_hour = time::duration_from_secs(3600);
             let one_hour_later_result = now.checked_add(one_hour); // If method style is used
             // ... handle Result ...

        },
        core::Result::Err(error) => {
             io::print("Error! Could not get system time: ");
             io::println(error.toString()); // Assuming the Error enum has a toString method
        }
    }

    // --- Sleep Function (Already shown in Thread module) ---
     thread::sleep(duration); // thread::sleep returns Result<void, time::Error>
}
```
Conclusion
The CNT Standard Library provides essential building blocks for your programs. By importing modules with the import statement and accessing their items with qualified names (module::Item), you can leverage the rich functionality offered by the Stdlib. It is especially important to use the core::Result type for functions that can fail and handle both success and error cases safely using the match expression.

This document is an introduction to using the Stdlib. To learn the full API of each module and the types it contains, you can refer to the corresponding .hnt (interface) files or future detailed documentation.