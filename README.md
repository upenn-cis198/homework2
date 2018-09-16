# homework2
A Rust Find utility

## Setup
No code will be given for this homework. Please create your own Rust Project using cargo. Name your project `rust_find`:

```bash
cargo new rust_find
```

## Assignment
We will be creating a command line utility similar to the Linux command `find`. That is, given a directory `root_dir` and a regular
expression pattern: `pattern`, your program will recusively go through `root_dir` and all it's sub-directories finding file
names which match `pattern`.

Example:
```bash
# ./rust_find <pattern> <root_dir>
./rust_find ".*\.rs" ./src
```

This command will return all files which end with `.rs` in the directory ./src and all sub-directories. We will use Rust's
regular expression engine, so the syntax for regular expressions may be different from what you're used to in other
languages or shells.

For example `".*\.rs"`, first `.*` matches any string of any length, then we escape the period
before the file extension: `\.` and finally we match `rs` (Note technically this could match files ending with say "rs.*"
but that's okay.)

So for simplicity, as long you're using the Rust Regex matcher you're doing it right.

This assignment is vague on purpose. Feel free to implement things as ou wish as long as you handle all errors. The answer to most questions you may have about the specification will be: Do something reasonable.

### Input
As input we receive 2 command lines argument from the user, a `pattern` and a `root_dir`. Any other number of arguments is an error. The pattern must be a valid Rust Regex, and root_dir must be a valid directory. Otherwise these are errors.

### Output
We print file paths, one per line, which match the `pattern` to stdout.
File path order does not matter, chose whichever order is easiest for you to implement.
There is many places where errors could occur, please exit the program and report the error gracefully, i.e. do not use panic or expect. All errors must be written to stderr.

## Design and implementation
Your code is expected to be separated into logical functions, each doing one task. Please use common sense software
engineering practices. Please see the `Grading Policy` section of the class website under `homework assignments`.

### Regexes
Please use the [Regex Crate](https://docs.rs/regex/1.0.5/regex/) for regular expressions.

### Paths
The `Path` type represents OS paths/files/directories, this type is unsized, meaning it is not adequate to pass around
functions on it's own. Instead we will use `PathBuf` for our functions.

You may see `&Path` or consider the type signature signature for `read_dir`:
```rust
pub fn read_dir<P: AsRef<Path>>(path: P) -> Result<ReadDir>
```
This functions can be understood as "takes a `P` as long as `P` can be turned into a Path reference (`P: AsRef<Path>`).
Even though we haven't discussed this feature, pretty much anything that looks like a path will work: PathBuf, String, &str, etc. As long as you pass it by reference.

Similarly: PathBuf implements a `Deref<Target=Path>` meaning `PathBuf` can be thought of a pointer/reference be converted to Path, so any method on `Path` will work on `PathBuf`. Do not worry, we will cover this feature later. You can see
the methods "inherited" from Path [here](https://doc.rust-lang.org/std/path/struct.PathBuf.html). So if you have a PathBuf:
```rust
let pathbuf: PathBuf = ...;
pathbuf.is_dir(); // Just works
```
### Functions to use.
In general, any function on the standard library is fair game to use. You may only use external crates specified in the
homework. Do NOT use any other external crates. Feel free to use functions from external crates that are specified in
the assignment however.

Of interest to you, may be the following functions:
```rust
std::env::args()
std::fs::read_dir()
Regex method is_match()
Path/Pathbuf method to_string_lossy()
Vector method extend() and push()
```

### Dealing With Errors:
Many of the filesystem functions we will be using return Result<_>, this can quickly get out of hand. Please use the Rust `?` syntax for propagating errors. I have the following structure for my program which is recommended but not mandatory.

```rust
// Result is aliased to std::io::Result for all these functions.

/// Runs program, handles errors and returns.
main() -> Result<()>


/// Iterate through file paths, returning only those that match our pattern.
/// Notice that this fuction cannot error.
fn get_matches(paths: Vec<PathBuf>, pattern: Regex) -> Vec<PathBuf>


/// Recurse through the directory structure returning all files in all directories.
fn get_directories(dir: ReadDir) -> Result<Vec<PathBuf>>


/// Given the command line arguments, return a regex and the path to search through.
/// On error, prints message to stderr and returns None.
/// Errors: Not enough or too many arguments.
///         Bad regular expression pattern.
fn parse_args(args: Vec<String>) -> Option<(Regex, PathBuf)>
```

### Testing
Testing programs that interact with the Operating Systems is far more difficult than testing pure code. Therefore, I recommend
separating your pure code (code that does not interact with the OS) with code that does. That is, instead of having a single
function which iterates over the directory substructure, and does the regular expression matching. I recommend two functions,
one function which returns the directory structure as a list (`f1`), and another function that matches `pattern` on element of that
list (`f2`). This way, you can create simple unit tests for `f2`.

Testing is a tool, you should stop testing when you feel confident with your implementation and tests.

## Submisions
Pleas submit this assignment though github class room. This assigment will be due on Wednesday September 26.
