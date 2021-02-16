# homework2

A Rust find utility

## Setup

No code will be given for this homework. Please create your own Rust Project using cargo. Name your project `rust_find`:

```bash
cargo new rust_find
```

## Assignment

We will be creating a command line utility similar to the Linux command `find`. That is, given a directory `root_dir`, your program will recursively search `root_dir` and all its subdirectories to find files which match a set of given criteria.

I have the following 3 goals in mind for this assignment:

1. Working with a good data model (defining a `struct` to represent files) (see **Data Model** below)

2. Handling errors robustly (e.g., don't just crash if you can't read a file or directory) (see **Handling Errors** below)

3. Basic interaction with the operating system (file input, system calls, and command line arguments with `structopt`)

As always, please ask on Piazza if you get stuck or blocked! There are no stupid questions -- you can always ask anonymously.

### Overview

The user runs your program and specifies some criteria (command line arguments) for which files they want to find.
The criteria that you choose to allow will be up to you; I would like you to be creative in deciding what data model you want to use for files. For example, you could allow the user to match only files, only directories, only files whose size is above a certain value, or whose names contain special characters.
However, you should at least support returning all files whose name matches a regular expression.
Example:
```bash
> cargo run -- -p ".*\.rs" -d .src/
../temp/homework1/src/lib3.rs
../temp/homework1/src/lib2.rs
../temp/homework1/src/lib.rs
../homework1/src/lib3.rs
../homework1/src/lib2.rs
../homework1/src/lib.rs
../rust_find/src/main.rs
```

Notice that we call `cargo run` with the separator `--` between our programs arguments. This is needed to disambiguate the arguments to cargo
from our arguments.

This command will return all files which end with `.rs` in the directory `./src` and all sub-directories. We will use Rust's
regular expression engine, so the syntax for regular expressions may be different from what you're used to in other
languages or shells.

For example `".*\.rs"`, first `.*` matches any string of any length, then we escape the period
before the file extension: `\.` and finally we match `rs` (Note technically this could match files ending with say `"rs.*"`
but that's okay.)

So for simplicity, as long you're using the Rust Regex matcher you're doing it right.

This assignment is a bit vague on purpose. You are welcome to take some liberties with the design, as long as you support all the necessary functionality.

Note in the following example, we can also match at the directory structure, that is, "any file ending with `.rs` under a `src/` directory:

```bash
>>> ./target/debug/rust_find  "src/.*\.rs" ../
../temp/homework1/src/lib3.rs
../temp/homework1/src/lib2.rs
../temp/homework1/src/lib.rs
../homework1/src/lib3.rs
../homework1/src/lib2.rs
../homework1/src/lib.rs
../rust_find/src/main.rs
```

### Command Line Arguments

We will generate our command line input interface using the [StructOpt Crate](https://docs.rs/structopt/0.3.21/structopt/).
To use a crate, you add it to your `Cargo.toml` file, under `[dependencies]`, like this:

```
[dependencies]
structopt = "0.3.21"
```

For a simple example of how to use `StructOpt` that you can start from, check out [the demo from lecture 1](https://github.com/upenn-cis198/lecture1/blob/master/demos/src/bin/structopt.rs).

StructOpt should automatically generate an interface for you when you run `cargo run ... --help`.
Here is an example of what it might look like:
```bash
rust-find 0.1.0
A command line utility for searching for files with regexes

USAGE:
    rust_find [FLAGS] [OPTIONS] --dir <dir> --pattern <pattern>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -d, --dirs <dirs>                        List of directories to search in.
    -p, --patterns <patterns>                List of patterns to use.
    -o, --output <output>                    Write results to output file instead of stdout.
    -s, --size <size>                        Match files above size <size>.
```

**Hints:**

- StructOpt allows you to specify arguments as optional by making their type
an `Option<T>`.

- For arguments that should be a *list* (`--dirs` and `--patterns`),
  StructOpt should allow you to specify this by making the type a `Vec<T>`.

- You can support default values for your arguments using the `default_value = ` attribute. See the StructOpt documentation for some more examples of attributes.

- `patterns` and `dirs` should be required (not optional). The exact set of flags and options you allow is up to you, but there should be at least 2 things other than `-d`, `-p`, and `-o` -- see "Data Model" below.

### Data Model

A main part of this assignment is that I would like you to think about a data model for files. Please define some kind of `struct` in your code that represents a file, e.g.

```Rust
struct MyFile {
    name: String,
    dir_in: String,
    size_bytes: u64,
}
impl MyFile {
    // ...
}
```

You should define a function, such as `MyFile::from(path: &str)` which initializes your file data type from a file path. (This will require interacting with the system to get information about the file).
Then, use this to implement **at least two** command line criteria that the user can use other than just giving a regex pattern for the file name; file length was one such example, but you can also pick something else, like whether the file name contains a special symbol.

To be specific: please support at least the `-p --patterns`, `-d --dirs` and `-o --output` flags.
Then support **at least two** of the following:

- Matching on file size (in bytes, kilobytes, megabytes, etc.)

- Matching files which do/do not contain a special character

- Matching files of at most a certain depth (e.g., search only 3 levels of nested directories)

- Matching directories in addition to files

- Matching files with a given file type

- Matching files with certain permissions set

- A different criteria that you want to come up with!

### Output

We print file paths, one per line, which match the `--pattern` to stdout. Unless the `--output`
option is specified, then the results are written to the specified file.

File path order does not matter; choose whichever order is easiest for you to implement.

### Error Handling

There are many places where errors could occur.
Part of this assignment is to practice good error handling hygiene.
In partiular, do **not** crash on an error unless it is fatal: for example, if you
can't read from a file, or can't access it, you should **report a warning**
for that file and move on to the next one.
Here are some specifics:

- If a pattern in the patterns-file is malformed, we print a warning about the malformed pattern,
    and continue processing the rest of the patterns.
- If some dir in the dirs-file results in an error (e.g. is not a dir, does not exists, etc), we
    we continue processing the rest of the dirs.
- If a file or directory is not readable or you encounter some error on trying to get information about it, report a warning and continue to the next file or directory.

Many of the filesystem functions we will be using return `Result<_>`; the easiest way to deal with this is to write your own function to return `Result<_>` itself, and then use the Rust `?` syntax to propagate the error upwards.

For example, here are some functions you could write:

```rust
/// Iterate through file paths, and initialize a set of MyFile objects
/// Notice that this function does not error, but internally it will print
/// warnings if any paths don't work or if something goes wrong for a
/// particular file.
fn get_files(paths: Vec<PathBuf>) -> Vec<MyFile>

impl MyFile {
    /// Initialize a MyFile from a path
    /// This function can error, so it returns a Result object
    fn from_path(PathBuf) -> Result<Self> {
        // ...
    }
}
```

## Design and implementation

Your code is expected to be separated into logical functions, each doing one task. Please use common sense software
engineering practices. Please see the `Grading Policy` section of the class website under `homework assignments`.

### Regexes

You can use the [Regex Crate](https://docs.rs/regex/1.0.5/regex/) for regular expressions.

### Path Type

Here are some notes on how to handle file paths when you are searching over files.
The `Path` type represents OS paths/files/directories, this type is unsized, meaning it is not adequate to pass around
functions on it's own. Instead we will use `PathBuf` for our functions.

You may see `&Path` or consider the type signature signature for `read_dir`:
```Rust
pub fn read_dir<P: AsRef<Path>>(path: P) -> Result<ReadDir>
```
This functions can be understood as "takes a `P` as long as `P` can be turned into a Path reference (`P: AsRef<Path>`).
Even though we haven't discussed this feature, pretty much anything that looks like a path will work: PathBuf, String, &str, etc. As long as you pass it by reference.

Similarly: PathBuf implements a `Deref<Target=Path>` meaning `PathBuf` can be thought of a pointer/reference be converted to Path, so any method on `Path` will work on `PathBuf`. This is similar to how `&String` gets automatically understood as `&str`, but don't worry, you don't need to understand the details. You can see
the methods "inherited" from Path [here](https://doc.rust-lang.org/std/path/struct.PathBuf.html). So if you have a PathBuf:
```rust
let pathbuf: PathBuf = ...;
pathbuf.is_dir(); // Just works
```

### Functions to use

In general, any function on the standard library is fair game to use. You may only use external crates specified in the
homework. If you want to use any other external crates, though, please ask first.

Of interest to you, may be the following functions:
```rust
std::fs::read_dir()
Regex method is_match()
Path/Pathbuf method to_string_lossy()
Vector method extend() and push()
```

### Testing

Testing programs that interact with the Operating Systems is far more difficult than testing pure code. Therefore, I will not require that you have unit tests everywhere, but you should have some for the parts of your code that don't involve the operating system.
You may find it cleaner to separate your pure code (code that does not interact with the OS) with code that does. For example, separate the part of your code that initializes `MyFile` objects from the parts which match `MyFile` objects against the patterns provided by the user.

Testing is a tool, you should stop testing when you feel confident with your implementation and tests.

## Submissions

Please submit this assignment though GitHub Classroom. This assignment will be due on Thursday, March 4th (11:59pm Eastern).
